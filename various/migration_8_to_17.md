### 1. Add-opens

--add-opens
java.xml/com.sun.org.apache.xerces.internal.jaxp=ALL-UNNAMED
--add-opens
java.base/java.lang=ALL-UNNAMED
--add-opens
java.base/java.security=ALL-UNNAMED
--add-opens
java.base/java.io=ALL-UNNAMED
--add-opens
java.base/java.util=ALL-UNNAMED
--add-opens
java.base/java.lang.reflect=ALL-UNNAMED
--add-opens
java.base/java.text=ALL-UNNAMED
--add-exports
java.desktop/sun.awt=ALL-UNNAMED
--add-exports
java.naming/com.sun.jndi.ldap=ALL-UNNAMED
--add-exports
java.xml/com.sun.org.apache.xerces.internal.jaxp.datatype=ALL-UNNAMED
--add-opens
java.xml/com.sun.org.apache.xerces.internal.jaxp.datatype=ALL-UNNAMED
--add-opens
java.base/java.lang.invoke=ALL-UNNAMED
--add-opens
java.base/java.time=ALL-UNNAMED
--add-opens
java.base/java.util.concurrent=ALL-UNNAMED
--add-opens
java.base/java.lang.invoke=ALL-UNNAMED
--add-opens
java.management/javax.management=ALL-UNNAMED
--add-opens
java.naming/javax.naming=ALL-UNNAMED

### 2. Upgrades

- jaxb-impl
- mockito
- resteasy
- spring
- camel
- 