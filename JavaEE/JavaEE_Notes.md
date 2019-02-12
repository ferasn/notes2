JCA adapter activation specification:  Binding to specific adapter


Each JCA adpater inbound flow has a set of properties defined for its activation specification bean.  In MDB, those properties are defind in MDB annotations or in deployment
descriptor:

<activation-config>
	<activation-config-property>
		<activation-config-property-name>1111</activation-config-property-name>
		<activation-config-property-value>2222</activation-config-property-value>
	</activation-config-property>
	<activation-config-property>
		<activation-config-property-name>XXXX</activation-config-property-name>
		<activation-config-property-value>YYYY</activation-config-property-value>
	</activation-config-property>
</activation-config>

When deploying the MDB, the container needs to know the adapter against which to activate the MDB and supply these properties to construct its activation specification.
In WAS, this done by defining an activation specification under a JCA adapter, which is just a configuration of properties similar to the ones supplied by the MDB. Upon deploying the MDB, it is bound to one of these activation specification configuration objects. This way the adapter for which the MDB to activate is identified. In glassfish,
this is done by supplying a glassfish deployment descriptor "glassfish-ejb-jar.xml" that contains the resource adapter name in "resource-adapter-mid" element:

https://docs.oracle.com/cd/E18930_01/html/821-2418/beama.html.


For JCA outbound flow, there is no problem identifying the adapter because a connection is obtained through a JCA connection factory administered object defined and bound in JNDI for a specific adapter.





-------------------

* namespaces:
	* private namespace (component environment): per compoenent java:comp/env
	* global JNDI namespace: vendor implementation dependent
	* portable JNDI namespace: java:[scope]/ .  In WAS, java:global naming context pints to cell/applications in WAS global JNDI namespace.  e.g. a datasource "ds" bound under java:global/env/jdbc/ds will be under cell/applications/env/jdbc/ds.
		In Java EE under WAS, if a lookup for a resource under java:global (e.g. java:global/env/jdbc/ds) is not found it is reported in the log as "Context: cell/applications, name: env/jdbc/ds: First component in name ds not found".  This error in the log assums that the contexts env/jdbc already exist (another resource is bound under it that caused it to be created).
	https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/cnam_name_space_partitions.html
		
* In WAS, if a Resource annotation has attribute "lookup", during installation the mapping of the resource reference to a resource in the global JNDI namespace is optional. If a mapping is specified, it overrides the value in lookup. "mappedName" attribute does not seem to have
		any effect in WAS and appears to me that WAS ignores it.  Even in the WAS documentation in knowledge center there is nothing mentioned about this attribute.  This attribute is optional to implement according to javadoc:
			"Application servers are not required to support any particular form or type of mapped name, nor the ability to use mapped names"
		Reference: https://docs.oracle.com/javaee/6/api/javax/annotation/Resource.html
  "lookup" attribute of Resource annotation in WAS defaults to server root context, so a lookup of "jdbc/ds" will be looked up under server root context which is also the default for any defined resource in WAS Admin console and also the intitial context for any Java EE Application.
 
* In WAS Admin console, you can bind a resource (DataSource, JMS Connection Factory) to "java:global" namespace. In this case the resource will created under "cell/application" context

* "name" attribute of Resource annotation creates a reference for the resource in a namespace. By default, the namespace is the private component environment namespace java:comp/env.  You can also, specify one of java:[scope]
	namespaces and an entry will created there for the resource.  However, whether it is private namespace or java:[scope] namespace, a mapping to a defined resource in global JNDI needs to specified during installation for the reference.
	Once the mapping is specified, the entry in private namespace or java:[scope] namespace will be pointing to that resource.

* the "name" attribute of "Resource" annotation can be:
	* relative name: will be declared as subcontext of component environment java:comp/env.  e.g. "jdbc/myDB" will default to "java:comp/env/jdbc/myDB"
	* qualified JNDI: java:module/env , java:app/env , java:global/env. e.g. "java:app/env/jdbc/myDB"
	* no name provided: a default name will be provided based on the field and class name and will declared a subcontext of component environment : java:comp/env
	
	
* "beanName" attribute of EJB annotation (within the application): Only applicable if the target EJB is defined within the same application or stand-alone module as the declaring component.  Cannot be used with "lookup" attribute.
* "lookup" attribute of EJB annotation (across applications): was introduced to provide a portable way to reference EJB components in other applications. the string provided is the portable JNDI java:[scope].  
	Note: this same attribute exists for Resource annotation and while the one for Resource accepts any global JNDI name, the one for EJB only accepts portable JNDI names java:[scope].  This is indeicated in javadoc of both:
	https://docs.oracle.com/javaee/6/api/javax/annotation/Resource.html
	https://docs.oracle.com/javaee/6/api/javax/ejb/EJB.html


resource-env-ref is a reference to a general managed object. resource-env-ref was used prior to J2EE 1.4 to declare JMS destinations.
In J2EE 1.4 and later, JMS destinations are declared in message-destination and message-destination-ref elements.
Both can be used to reference JMS administered objects but message-destination-ref has more elements.

Reference: https://www.ibm.com/support/knowledgecenter/en/SSAW57_9.0.0/com.ibm.websphere.nd.multiplatform.doc/ae/tejb_mdb.html


Java EE application resource declarations:
------------------------------------------
https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/cejb_ref.html

EJB annotations:
https://stackoverflow.com/questions/10889563/ejb-3-1-localbean-vs-no-annotation


How WAS resolves references automatically:
--------------------------------------------
https://developer.ibm.com/answers/questions/422390/how-ejb-link-auto-link-works-in-websphere-applicat/


------------------
* LTC:
	* JDBC Connection and JMS Session (also applies to to any other JCA connection):  A transacted JMS Session ( createSession(Session.SESSION_TRANSACTED)) or a JDBC Connection (setAutoCommit(false)) is always involved in a transaction.  
		All JMS Session send and receive or JBDC SQL issued is part of the transaction. As soon as the commit or the rollback method is called, one transaction ends and another transaction begins. 
		Closing a transacted JMS Session or a JDBC Connection rolls back its transaction in progress, including any pending sends and receives or SQL changes.
	* In the absence of a global transaction, the JDBC Connection or JMS Session are in local transaction mode and if you follow get-use-close pattern of connection usage, the close at the end will cause the active transaction 
		to rollback unless you explicitly commit() before close().  In order to provide a unified programming experience, WebSphere starts LTC in the absence of a global transaction.  This LTC will eventually commit or rollback (If the application returns control to the container by an exception) the 
		active local transaction of any JMS Session or JDBC Connection under its scope.  Therefore, an application can follow a get-use-close pattern of connection usage, regardless of whether the application runs in a global transaction. 
		The application can depend on the close action behaving in the same way in all situations, that is, the close action does not cause a rollback to occur on the connection if there is no global transaction.
	* You can still commit or rollback the local transaction of JMS Session or JDBC Connection explicitly under LTC. The LTC just defines the boundary at which all RMLTs (resource manager local transactions) must be complete; any incomplete RMLTs are resolved, according to policy, by the container.
	
	Reference: 
		* https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/cjta_loctran.html
		* https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/rjta_useltran.html
		* http://www-01.ibm.com/support/docview.wss?uid=swg27011067&aid=1
* JCA specs does not allow sharing of non-transaction resources.  Since JMS connection for MQ adapter is non-transactional, it cannot be shared.  It the JMS Session of MQ adapter that is transactional and therefore,
shareable by default.  The JMS connection for the SIBus is transactional and therefore is shareable.
* Why JMS for MQ has a pool for JMS Session per connection?  it is related to the JMS connection being non-transactional and after it is closed it is returned to the free pool.  The session created from that connection
will not be returned to the free pool even if it is closed until the transaction it is enlisted under is finished.  Hence, this connection can be picked by another thread while the session created from this same connection is busy
by a transaction in the prior thread.  If there was no pool for the JMS Session, the new thread will block waiting for the Session to become free.  This scenario cannot happen with SIBus JMS Connection because it is transactional,
and will not be returned to the free pool after close() until the transaction ends and therefore cannot be picked by a different thread.
* A a rule of thump, a JCA connection (also , JMS Session) will not be returned to the free pool after close in the following scenarios:
	1- The connection is non-transactional.
	2- The connection is transactional but it is managing it is own transaction under LTC, and its sharing property is set to unshareable.  If it is managing its own transaction (starting a local transaction and committing it),
	but it is shareable, LTC will not let go to the free pool, until LTC context ends.  In case, it is closed and a connection is created from the ConnectionFactory, a handle to the same physical connection is returned,
	because it is set to be shareable.
* Java EE specs: one "active" JMS session per component.  You can create another JMS session after closing the active JMS session.
* JDBC and JMS when used in Java EE container environment (Under global transaction), certain methods are not supported (e.g: JDBC connection setAutoCommit).  The Java EE specs details these.  However, in LTC, 
	these methods are supported (e.g. JDBC connection setAutoCommit).
* In a container when you create a connection from a connection factory, you don't get the actual physical connection object but a handle to the physical connection.
* Global Transaction and LTC:
	* transaction containment: at the end either commit or rollback connections
	* Context for sharing: Shareable transactional resources.  : Under LTC: if a connection is configured to be unshareable, after close(), the connection is returned to the free pool.  If it is shareable, it will not be returned
	to the free pool until transactional context LTC ends.
	
* WebSphere MQ JMS Adapter in WAS:
	* Connection: is lightweight and non-transactional and therefore non-shareable.
	* Session: represents the actual MQ connection and it is transactional and therefore Shareable by default.
	
	

**** Extra Resources to look at:
https://www.ibm.com/developerworks/community/blogs/aimsupport/entry/j2ca0086w_ltc_and_running_out_of_shared_connections_in_websphere_application_server?lang=en
https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.multiplatform.doc/ae/cdat_codatc.html
https://www.ibm.com/developerworks/community/blogs/aimsupport/entry/Common_Reasons_Why_Connections_Stay_Open_for_a_Long_Period_of_Time_in_WebSphere_Application_Server?lang=en
