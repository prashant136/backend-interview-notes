Assume karo Bob (dev) ek spring app { windows, java 25, mysql 8, mongo 6 } pe kaam kr rha hai.
now Alice take this app try to deploy this app in QA env { linux, java 17, mysql 7, mongo 5 }. 
due to version and env mismatch app will fail to deploy in QA env.
now another dev Charlie { macOS, java21, mysql 7, mongo 8 } try to change something in this codebase. but app is not not working again in Charlie's local machine because of env mismatch.
this type of environment mismatch generally happens. 
**Docker solve this environment mismatch.**


Docker is a tool that allows you to package your app and its entire environment into one unit.

**Docker Image** - A docker image is a packaged template of your application. 
It contains -
	application Code, java runtime, libraries, OS package, config files. 
	(codebase + runtime + dependencies + configuration to start app) 
	by using this image docker can run at different instances.

**Docker Container** - A container is a running instance of an image. we can run different container by using same image on any environment or any machine.

Now bob creates an image of application { windows, java 25, mysql 8, mongo 6 } and share this image with charlie an alice.  now charlie & alice both run this image inside docker container in his machine. because this docker image can run any machine irrespective of the env of the machine (image will not depend on machine's local env)

**Docker architecture -**
Now Bob need something interact with docker. bob needs to install docker on his machine. when you install docker on your machine, you'll get docker CLI (where you can write command in order to interact with docker and manage docker orchestration). OR we'll also have an option docker desktop (alternate to docker CLI). I will not recommend docker desktop rather use docker CLI because you won't get docker desktop in cloud and other environment.
Docker daemon (dockerd) (docker daemon background process hai) this is brain and core of docker which is executing the command (jo hum docker CLI me likhte hai). if daemon isn't running inside env. your queries won't work. docker daemon has different responsibilities -
- Containers ko start, stop, pause remove karna.
- Docker Hub se images pull/push karna aur build karna.
- Containers ke data volumes aur storage drivers manage karna.
- Containers ke beech virtual network networks, IP addresses, aur ports manage karna.

Now Bob create an image. How will bob share image with charlie/alice ?
**Docker hub**

docker pull <?something?> = docker daemon this particular image isn't available at local, so pull it form docker hub to machine.

---------------------------------

**Installation :-**

**In order to start docker daemon, we need to start docker desktop. If docker daemon isn't running then docker's command won't execute.**


```java

--help ==> --help flag to get context specific documentation directly in your terminal.
	docker --help ==> This displays the top level documentation for Docker.

	git --help ==> this gives you the entry point documentation for Git.

	docker ps --help ==> This filters out everything else and drills down _only_ into the 'ps' command.
	


####### ------- some docker commands ------- #######

1.) docker ps - sirf active containers ki list show karega
	docker ps --a  - show all conatiners
	
2.) docker pull <name>  - downloads container images from a registry (Docker Hub) to your local machine

3.) docker images (show list of all images locally)

4.) docker run [OPTIONS] IMAGE [COMMAND] [ARG...] -
if docker image locally nhi toh usee pull karega phir new conatiner banyega use run karega. (brand new fisrt time)
if image is present then simple satrt container from image.

5.) docker run nginx (Foreground Mode) - current terminal window container se attach ho jayega. terminal me koi nayi command execute nahi kar sakte jab tak container chal raha hai. ctrl+c se exit krne p conatiner stop ho jayega. (debugging, testing mode)

6.) docker run -d nginx (Detached Mode) - 
- Pehle ek Brand New Container Create hota hai, (Hash ID termonal pe print hogi) fir background me start hota hai. Naya container blank state ke saath banta hai (Puraane instance se koi lena dena nahi). Har baar command chalane par ek Nayi Container ID generate hogi.

7.) docker stop containerID -- perticlar container stop ho jata hai.

8.) docker start <container_id> -- Puraane Stopped/Existing Container ko restart karta hai.

9.) docker rm <container_id_or_name> -- delete single container
 {conatiner ko stop kro phir delete kro }

10.) docker rmi <image_id_or_name> -- delete single image
		rm - remove containers
		rmi - remove images
		
11.) docker run -it <image> -- conatiner run hoga terminal pe interactive mode me. and 'exit' command se exit hoga.

12.) docker container prune -- Saare Stopped containers ek sath delete karega. (running containers ko nhi)

13.) docker build -t grocessyApp:1.0.0 . - (docker image create krta h tag name aur version) { Do not forget the trailing dot ('.'), which tells Docker to use the current working directory as the build context }

14.) docker run -p host_port:container_port image_name 
(port mapping)

15.) docker exec -it <conatiner_ID> bash - pehle se chal rahe (running) container ke andar enter hone aur commands run karne ke liye kiya jata hai.

16.) 🚩 docker run -d \
  -e SERVER_PORT=9090 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:postgresql://postgres-db:5432/mydb \
  -p 8080:9090 \
  spring-demo:1.0 (docker enviornment variable)
  
17.) docker run --name <custom_container_name> <image_name> - (assign custom name to a Docker container)
  

[[[ Docker me rule yeh hai ki pehle container delete karenge, uske baad hi image delete hogi. ]]]
[[[ Stopped container hi delete honge, running container nhi ]]]

```

--------

**Docker image for Spring App -**

Why Dockerfile ?  **Docker needs clear instructions to create the image.**

Without Dockerfile,  Docker does not know :
	Which runtime to use?
	Where to put code?
	How to install dependencies?
	Which command starts the app?
	Which port app uses?

```
---- dockerfile --------

# 1. Base Image: OpenJDK 17 Runtime Environment
FROM eclipse-temurin:17-jdk-alpine

# 2. Working Directory in Container
WORKDIR /app

# 3. Build ki hui JAR file ko container me copy karein
# Target folder se jar file uthakar container ke inside app.jar naam se paste karein
COPY target/*.jar app.jar

# 4. Container kis port par listen karega
EXPOSE 8080

# 5. Container start hone par execution command
ENTRYPOINT ["java", "-jar", "app.jar"]   
```

```
----  ab locally image banke and containe run karenge ---
1. app ka JAR build
./mvnw clean package -DskipTests

2. docker image build karo
docker build -t spring-demo:1.0 .

3. container run karo
docker run -d -p 8080:8080 spring-demo:1.0   
```

🚩 but docker run -d spring-demo se container run krte hai. then postman pe api hit karne pr error aata hai. because application is running inside container 8080 port, postman sends to 8080 port my local MacBooks 8080.
postman container ke andar port se map nhi ho pa rha hai. isisliye hum docker me port mapping krte hai.

docker run -p  host_port : container_port  image_name

----------

**🚩 Environment Variable -**
Code changes with Features, environment Variables change with Deployment Targets. har environment ke liye dynamically values set hongi. (code me changes nhi honge), docker me bhi env. ke according dynamic value set hoti hai.

---------------
**🚩 Docker Volumes -**
_Volumes are persistent data stores for containers_, created and managed by Docker. 
Docker containers are ephemeral, meaning data stored inside a container is lost when the container is removed.

agar hum ek image ke multiple containers bhi bnaye phir unke across data persistent rhati hai. 
```java
			   ┌─────────────────┐
			   │  Docker Volume  │
			   └────────┬────────┘
	                (File Sharing)
	            /         │         \
	           /          │          \
	          ▼           ▼           ▼
	    [Container]  [Container]  [Container]


1.) docker run -d \
  -v nameVolm:/app \         ( Named Volume Mount )
  -e SERVER_PORT=9090 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -p 8080:9090 \
  spring-demo:1.0

-v <volume_name>:<container_inside_path>
(host machine par 'nameVolm' naam ka Docker Volume banata hai aur usko container ke andar '/app' directory se connect kar deta hai. Container stop ya delete bhi ho jaye, tab bhi '/app' folder ka data 'nameVolm' me safe rahega.)

2.) docker volume ls (list of volumes)

3.) docker volume inspect nameVolm (inspect volume details)

4.) docker volume rm nameVolm (delet volume)


[[[ container dleete krne se volume delete nhi hota hai ]]]
[[[ pahle container delete karo phir volume ]]] 
```

Docker me Bind Mounts (-v /host/path:/container/path) aur Named Volumes (-v vol_name:/container/path) me kya difference hota hai?

-----------------

**✅ Docker Compose -** docker compose allows us to define and run multi container docker application using a YAML file.

```java

1.) docker compose up => Ye docker compose.yml file ko padh kar usme defined saare containers, networks, aur volumes ko create aur start karta hai. ye forgound mode me up hota hai. Ctrl + C me stop ho terminate ho jata hai.

2.) docker compose up -d => similer but (detached mode) yahi background me run hota rhta hai.

3.) docker compose down => Ye docker compose.yml file mein chal rahe saare containers ko band karke delete kar deta hai aur network bhi cleanup krta hai. but named volumes ko delete nhi krta (data safe rhta hai).

4.) docker compose ps => sirf running containers ki list

5.) docker compose ps => saare available containers ki list

6.) docker compose up --build => Agar Dockerfile ya app me koi change kiye hai, purane image use karne ke bajaye ek naya image banaye (rebuild kare). 
then '--build' flag use karo.

7.) docker compose down -v => Isme containers ke sath volumes ko bhi delete krta hai.

```


-------

**🚩 Docker Networking -**  you have one container where spring boot app is running and another container where MySQL is running. how these two containers will communicate between each
other.  in order to solve this problem we register them into a same network, so docker network come into picture.

```

```



-------------------------
-----------

✅ Important for spring boot --
- mvn: Maven (System Level).
- ./mvnw: Maven Wrapper (Project Level Script).
- 👉 **Best Practice:** Production, Dockerfiles, aur team projects me hamesha `./mvnw` use karein taaki sabhi machines par bina kisi build mismatch ke kaam ho sake.

**Maven ki Build Lifecycle phase me kaam karti hai:**

validate  ⟶  compile  ⟶  test  ⟶  package  ⟶  verify  ⟶  install

```java
[Clean Phase] -> Target folder delete karta hai.
      ↓
[Compile Phase] -> Java files (.java) ko byte code (.class) me compile karta hai.
      ↓
[Test Phase] -> Unit tests run karta hai.
      ↓
[Package Phase] -> Compiled code ko .jar / .war file banakar `target/` folder me daalta hai.
      ↓
[Install Phase] -> `target/` folder wali .jar ko local repository (`~/.m2/repository`) me copy karta hai.
```


`mvn clean package` : 

    Pure project ko clean aur compile karta hai.
    `target/` directory me `.jar` file generate kar deta hai.
    Use Case: Jab aapko sirf runnable JAR file chahiye ho Dockerize karne ya server par deploy karne ke liye.
        
`mvn clean install` :

    package wala saara kaam karta hai (JAR file `target/` me banata hai). aur us  .jar file ko tumhare Mac ke **Local Maven Repository** (`~/.m2/repository`) me copy/install kar deta hai.

`mvn clean install -DskipTests` => test case skip krke build ready karo.
