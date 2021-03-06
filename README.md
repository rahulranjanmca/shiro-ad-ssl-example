# shiro-ad-ssl-example
====================

 An example of using Shiro to perform Active Directory authentication over SSL.

This simple example illustrates how you can configure Shiro via the shiro.ini to do authentication and authorization using Active Directory over SSL. It uses Shiro's ActiveDirectoryRealm for authc/authz, and Shiro's JndiLdapContextFactory for the connections to AD. There are a few steps to be performed before you can run the LdapSslExample#main method.

## Prerequisites
- An Active Directory server with some users and groups, configured to accept SSL connections.

## Instructions
 1. Modify src/main/java/shiro.ini, following the instructions within that file.
 2. Modify the userName and password variables of com.jhegg.reference.shiro.ldapSslExample.LdapSslExample. Those credentials will be used to authenticate. Note: this handling of credentials is insecure and in plain-text.
 3. If the Active Directory SSL certificate is not signed by a trusted CA, obtain and import the SSL certificate into your JRE or JDK cacerts keystore. (see the "Importing Self-Signed AD SSL Certificate" section below)
 4. Run LdapSslExample#main.

The output will indicate whether the user is authenticated, and whether it is authorized.

## Example shiro.ini
```properties
[main]
contextFactory = org.apache.shiro.realm.ldap.JndiLdapContextFactory
contextFactory.url = ldaps://ad.domain.com:636
contextFactory.systemUsername = shiro@domain.com
contextFactory.systemPassword = password
contextFactory.environment[java.naming.security.protocol] = ssl

realm = org.apache.shiro.realm.activedirectory.ActiveDirectoryRealm
realm.ldapContextFactory = $contextFactory
realm.searchBase = "CN=Users,DC=DOMAIN,DC=com"
realm.groupRolesMap = "CN=ShiroUsers,CN=Users,DC=DOMAIN,DC=com":"ShiroUsersRole"
```

Using the above example shiro.ini, if there exists some user (e.g., "jhegg") in the Users container of domain.com, then that user will be able to authenticate with this application. For example, the following code will execute without throwing an AuthenticationException:

```java
UsernamePasswordToken token = new UsernamePasswordToken("jhegg", "password");
Subject currentUser = SecurityUtils.getSubject();
currentUser.login(token);
```

And, if that user is a member of the ShiroUsers group (located in the Users container), then the following code will return true:

```java
currentUser.hasRole("ShiroUsersRole")
```

## Importing Self-Signed AD SSL Certificate
The easy way to obtain and import a self-signed Active Directory SSL certificate is as follows:
 1. Use openssl as a client, and dump out the certs.<br>
```
openssl s_client -connect ad.domain.com:636 -showcerts
```
 2. Copy the certificate block output (including the BEGIN and END lines) into a file such as adserver.crt.
 3. Import the SSL certificate into the JRE cacerts keystore, using an alias that represents the AD server (in case you need to update it at a later date). Note: the default cacerts keystore password is 'changeit'.<br>
```
keytool -import -keystore <path/to/jre/lib/security/cacerts> -alias <alias> -file adserver.crt
```

