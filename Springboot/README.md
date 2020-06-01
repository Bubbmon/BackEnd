# Springboot相关

##### ===================== updated on 2020-05-30 by Legion ======================

#### 1. 修改spring-boot-starter-parent中管理的版本

进入spring-bootstarter-parent中，再进入spring-boot-dependencies中可以看到springboot管理的各种依赖的版本号。2.1.7版本的mysql是8.x，如果想将其改为5.x可以用以下方式：

> ```xml
> <!-- 用于修改springframework中的mysql版本 -->
> <dependencyManagement>
>     <dependencies>
>         <dependency>
>             <groupId>mysql</groupId>
>             <artifactId>mysql-connector-java</artifactId>
>             <version>5.1.30</version>
>         </dependency>
>     </dependencies>
> </dependencyManagement>
> 
> <dependencies>
>     <!-- 由于更换了mysql版本，需要重新写依赖坐标 -->
>     <dependency>
>         <groupId>mysql</groupId>
>         <artifactId>mysql-connector-java</artifactId>
>     </dependency>
> </dependencies>
> ```



##### ===================== updated on 2020-05-31 by Legion ======================

#### 2. springboot在linux上的部署

##### 法一：打jar包后运行

打jar包上传至linux后，执行以下指令：

```shell
java -jar xxx.jar
```

##### 法二：压缩上传后解压，在linux上完成编译运行

这种情况通常要具备三个条件：

1. 含带@SpringBootApplication注解的启动类。

2. pom中有以下插件：

   ```xml
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

3. 在与pom同一路径的位置执行以下指令：

   ```sh
   mvn spring-boot:run
   ```

如果要进行后台运行，可以使用nohup [command] &的方式来运行。此时如果想要中止后台运行，可以使用kill的方法来停止进程。

经资料查询有更加安全的停止后台运行的方法，包含以下步骤：

1. 在pom中加入以下依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 在application.properties（或者application.yml，但格式不同）中写入以下配置：

   ```properties
   # 开启shutdown
   management.endpoint.shutdown.enabled=true
   # 公开所有端点
   management.endpoints.web.exposure.include=*
   # 自定义管理端点的前缀
   management.endpoints.web.base-path=/MyActuator
   # 自定义管理的端口
   management.server.port=12581
   # 只允许本机进行管理
   management.server.address=127.0.0.1
   ```

3. 编写相关的shell脚本

   ```shell
   #!/bin/bash
   PORT=$(netstat -ntulp | grep 8080) # 其中8080应更换为相应的服务器端口
   # 如果PORT不为空说明有程序占用该端口号
   if [[ -n $PORT ]]; then
           echo "The server is open, ready to close"
           RESULT="000"
           while [[ $RESULT == "000" ]]; do
                   echo "Closing down ..."
                   # 发送post请求关闭，12581是上面自己设置的管理端口
                   RESULT=$(curl -X POST 127.0.0.1:12581/MyActuator/shutdown)
                   sleep 2
           done
           echo "Close off success"
   fi
   ```

   

#### 3. springboot依赖冲突

目前发现在pom中手动写入Gson2.2.x的坐标后，编译时会出现以下报错：

> Failed to execute goal org.springframework.boot:spring-boot-maven-plugin

猜测可能是与springboot中的相关依赖有冲突，改为Gson2.6版本后即可成功编译。