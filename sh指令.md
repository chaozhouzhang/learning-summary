# sh
启动springboot的jar程序
```
#!/bin/sh

RESOURCE_NAME=beautify-0.0.2-SNAPSHOT.jar

 
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ]; then

echo 'Stop Process...'

kill -15 $tpid

fi

sleep 5

tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ]; then

echo 'Kill Process!'

kill -9 $tpid

else

echo 'Stop Success!'

fi

 
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ]; then

echo 'App is running.'

else

echo 'App is NOT running.'

fi

 
rm -f tpid

nohup java -jar ./$RESOURCE_NAME --spring.profiles.active=test &

echo $! > tpid

echo Start Success!
```

停止springboot的jar程序
```
#!/bin/sh

RESOURCE_NAME=beautify-0.0.2-SNAPSHOT.jar

 
tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ]; then

echo 'Stop Process...'

kill -15 $tpid

fi

sleep 5

tpid=`ps -ef|grep $RESOURCE_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ]; then

echo 'Kill Process!'

kill -9 $tpid

else

echo 'Stop Success!'

fi
```

