Spark requires HMS connection

```When you are executing a query on spark such as select * from <table>, it does have to contact with HMS to know the table location and other metadata. Without connecting to HMS its not possible for spark to execute the query. For it to authenticate itself to HMS it definitely needs to have either user credentials or a delegation token that oozie has obtained for the workflow. When you mention <action name="hive2-create-target-tables" cred="hs2-creds"> it indicates that the action is going to contact the hs2 service with the hs2-creds that oozie has obtained for the workflow. It doesn't mean that a shell action cant use the "hs2-creds" credentials. If the shell action has the requirement of contacting hs2 you must mention the respective creds it has to use in its action attribute.```

Without oozie the default behaviour of spark is mentioned in the below link:
    
```https://apache.googlesource.com/spark/+/master/docs/running-on-yarn.md#Running-in-a-Secure-Cluster```
```For a Spark application to interact with HDFS, HBase and Hive, it must acquire the relevant tokens using the Kerberos credentials of the user launching the application —that is, the principal whose identity will become that of the launched Spark application. This is normally done at launch time: in a secure cluster Spark will automatically obtain a token for the cluster's HDFS filesystem, and potentially for HBase and Hive.```

With oozie: 
 
```Apache Oozie can launch Spark applications as part of a workflow. In a secure cluster, the launched application will need the relevant tokens to access the cluster's services. If Spark is launched with a keytab(using the --keytab option), this is automatic. However, if Spark is to be launched without a keytab, the responsibility for setting up security must be handed over to Oozie.```

```In your case you are not passing --keytab option and hence the responsibility of getting credentials to HMS is delegated to oozie service now. Oozie will obtain the delegation token of hms only if you have explicitly request it in the workflow. For instance```

    <credentials> 
    <credential name="hs2-creds" type="hive2"> 
    <property> 
    <name>hive2.server.principal</name> 
    <value>${jdbcPrincipal}</value> 
    </property> 
    <property> 
    <name>hive2.jdbc.url</name> 
    <value>${jdbcURL}</value> 
    </property> 
    </credential> 
    </credentials> 

```The above section you mentioned in the workflow will make oozie to obtain delegation token for hs2 service. Similarly if you want to get the delegation token for HMS you need to add credentials of hcat type as mentioned in the link "https://oozie.apache.org/docs/4.2.0/DG_ActionAuthentication.html" Workflow Changes section ```
```And the reason for advising you to disable the property(spark.yarn.security.tokens.hive.enabled) is because we saw in [2] that spark by default will attempt to obtain delegation token from services it needs to communicate. Since oozie is taking care of getting the delegation token it is not advisable to make spark to get the delegation token again. To avoid Spark attempting —and then failing— to obtain Hive, HBase and remote HDFS tokens, the Spark configuration must be set to disable token collection for the services.``` 

