# nbmp
export TOMCAT_USER="tomcat"
export JAVA_OPTS="-Xms5144m -Xmx5144m -XX:NewSize=3096m -server -XX:+DisableExplicitGC -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -Dqunar.logs=$CATALINA_BASE/logs -Dqunar.cache=$CATALINA_BASE/cache -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_BASE/logs/gc.log"
export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
chown -R tomcat:tomcat $CATALINA_BASE/logs
chown -R tomcat:tomcat $CATALINA_BASE/cache
chown -R tomcat:tomcat $CATALINA_BASE/conf
chown -R tomcat:tomcat $CATALINA_BASE/work
chown -R tomcat:tomcat $CATALINA_BASE/temp

# market
export TOMCAT_USER="tomcat"
export JAVA_OPTS="-Xms3072m -Xmx3072m -XX:NewSize=1024m -XX:PermSize=256m -server -XX:+DisableExplicitGC -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -XX:+UseG1GC -Dqunar.logs=$CATALINA_BASE/logs -Dqunar.cache=$CATALINA_BASE/cache -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_BASE/logs/gc.log"
export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
chown -R tomcat:tomcat $CATALINA_BASE/logs
chown -R tomcat:tomcat $CATALINA_BASE/cache
chown -R tomcat:tomcat $CATALINA_BASE/conf
chown -R tomcat:tomcat $CATALINA_BASE/work
chown -R tomcat:tomcat $CATALINA_BASE/temp

# recommend
export TOMCAT_USER="tomcat"
export JAVA_OPTS="-Xms2048m -Xmx2048m -XX:NewSize=256m -XX:PermSize=256m -server -XX:+DisableExplicitGC -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -XX:+UseG1GC -Dqunar.logs=$CATALINA_BASE/logs -Dqunar.cache=$CATALINA_BASE/cache -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_BASE/logs/gc.log"
export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
chown -R tomcat:tomcat $CATALINA_BASE/logs
chown -R tomcat:tomcat $CATALINA_BASE/cache
chown -R tomcat:tomcat $CATALINA_BASE/conf
chown -R tomcat:tomcat $CATALINA_BASE/work
chown -R tomcat:tomcat $CATALINA_BASE/temp

# hc_backend
export TOMCAT_USER="tomcat"
export JAVA_OPTS="-Xms3072m -Xmx3072m -XX:NewSize=1536m -XX:PermSize=256m -server -XX:+DisableExplicitGC -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -XX:+UseG1GC -Dqunar.logs=$CATALINA_BASE/logs -Dqunar.cache=$CATALINA_BASE/cache -verbose:gc -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$CATALINA_BASE/logs/gc.log"
export JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
chown -R tomcat:tomcat $CATALINA_BASE/logs
chown -R tomcat:tomcat $CATALINA_BASE/cache
chown -R tomcat:tomcat $CATALINA_BASE/conf
chown -R tomcat:tomcat $CATALINA_BASE/work
chown -R tomcat:tomcat $CATALINA_BASE/temp