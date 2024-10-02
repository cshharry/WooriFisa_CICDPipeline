
# 🚀 Jenkins 실습

---

## 1. Jenkins 준비 🛠️

### 🖥️ **myjenkins 생성 및 실행**

```bash
docker run --name myjenkins2 --privileged -p 8080:8080 -v $(pwd)/appjardir:/var/jenkins_home/appjar jenkins/jenkins:lts-jdk17
```

![1](https://github.com/user-attachments/assets/d649671b-65bc-4a2c-aa83-7380cfb7e1e0)


### 🌐 **ngrok 실행**

```bash
ngrok http http://127.0.0.1:8080
```

![2](https://github.com/user-attachments/assets/810c1601-bcfe-4e5f-bc10-c1bc9f2a8298)
### 🔗 **GitHub 연결**

![3](https://github.com/user-attachments/assets/b273a1ff-65df-44f8-ad34-2bf62a4903fb)

### 🔌 **plugin - stage view 설치**

![4](https://github.com/user-attachments/assets/2363332c-f1c5-4788-bd63-7f7697b721dd)

### 🔐 **Jenkins 권한 부여**

```bash
docker exec -u root -it myjenkins2 bash
chown -R 1000:1000 /var/jenkins_home/appjar/
chmod -R 755 /var/jenkins_home/appjar
```

## 2. CI/CD 파이프라인 구축 - 스크립트 작성 📜

### 📂 **git Repo clone**
- GitHub 저장소에서 소스 코드를 Jenkins로 가져오는 단계로 Jenkins는 최신 소스 코드를 확보

```bash
pipeline {
    agent any

    stages {
        stage('Repo Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/cshharry/Jenkinson_Test.git'
                echo "다운로드 완료"
            }
        }
    }
}
```

### 🏗️ **Gradle을 활용한 Build**
- 소스 코드를 컴파일하고 패키징해 패키지 파일(JAR 파일) 생성

```bash
stage('Build') {
    steps {
        dir('./SpringApp') {  
            sh 'chmod +x ./gradlew'  
            sh './gradlew clean build -x test'
            sh "echo $WORKSPACE"  
        }
    }
}
```

### 🗂️ **JAR 파일 복사**
- JAR 파일을 Jenkins 서버의 특정 경로로 복사하는 단계

```bash
stage('Copy jar') { 
    steps {
        script {
            def jarFile = './SpringApp/build/libs/SpringApp-0.0.1-SNAPSHOT.jar'                   
            sh "cp ${jarFile} /var/jenkins_home/appjar/"
        }
    }
}
```

## 3. JAR 파일 실행 🚀

### 💻 **실행**

```bash
java -jar SpringApp-0.0.1-SNAPSHOT.jar
```

### 🌐 **App 서버 확인**

```bash
curl http://127.0.0.1:8999/test
```

![5](https://github.com/user-attachments/assets/cbb63619-bb98-4756-8424-22b180ad1e5d)
## 4. Jenkins를 활용한 원격 서버 SSH 접속 🖥️

---

### 🔐 **Spring Boot 애플리케이션**

- **배포 및 실행**

```bash
#!/bin/bash

# 변수 설정
JAR_FILE="SpringApp-0.0.1-SNAPSHOT.jar"
DEPLOY_DIR="/home/ubuntu/step07cicd"
BACKUP_FILE="$DEPLOY_DIR/SpringApp-0.0.1-SNAPSHOT.jar.bak"
LOG_FILE="$DEPLOY_DIR/app.log"

# 이전 JAR 파일 백업
if [ -f "$DEPLOY_DIR/SpringApp-0.0.1-SNAPSHOT.jar" ]; then
  echo "이전 JAR 파일을 백업합니다: $BACKUP_FILE"
  mv "$DEPLOY_DIR/SpringApp-0.0.1-SNAPSHOT.jar" "$BACKUP_FILE"
fi

# 새로운 JAR 파일 복사
if [ -f "$JAR_FILE" ]; then
  echo "새로운 JAR 파일을 복사합니다: $JAR_FILE -> $DEPLOY_DIR/"
  cp "$JAR_FILE" "$DEPLOY_DIR/"
else
  echo "JAR 파일을 찾을 수 없습니다: $JAR_FILE"
  exit 1
fi

# 기존 8999 포트 사용 중인 프로세스 종료
if sudo lsof -i :8999 > /dev/null; then
  echo "8999 포트를 사용하는 프로세스를 종료합니다."
  sudo kill -9 $(sudo lsof -t -i:8999)
fi

# 백그라운드에서 새로 실행
echo "Spring Boot 애플리케이션을 백그라운드에서 실행합니다."
nohup java -jar "$DEPLOY_DIR/SpringApp-0.0.1-SNAPSHOT.jar" > "$LOG_FILE" 2>&1 &

echo "배포 완료 및 애플리케이션이 실행 중입니다. 로그는 $LOG_FILE에서 확인할 수 있습니다."
```

### 🖥️ **스크립트 실행**

```bash
sh cicdbais.sh
```

### 🖱️ **원격 서버 SSH 접속**

```bash
stage('Run cicdbasic.sh on Host') {
            steps {
                script {
                    sshagent(['myjenkins-10.0.2.15-ssh-key']) {
                        // SSH를 통해 호스트에서 cicdbasic.sh를 실행
                        def host = 'username@10.0.2.15' // 호스트의 사용자 이름과 IP 주소
                        def shellPath = '/home/username/appjardir/' // cicdbasic.sh가 위치한 경로
                        sh "ssh -o StrictHostKeyChecking=no ${host} 'cd ${shellPath} && bash cicdbasic.sh'"
                    }
                }
            }
        }
```
### 🔌 **plugin - SSH Agent 설치**

### 🔑 **SSH 키 값 생성**

```bash
ssh-keygen -t rsa -b 4096 -C "이메일"
```

### 🔐 **SSH 공개키 원격 서버에 추가**

```bash
ls ~/.ssh
ssh ubuntu@10.0.2.15
```

### 🖱️ **SSH 자격 증명 추가**
- Dashboard → Jenkins 관리 → Credentials → Add Credentials
- Private key는 `cat ~/.ssh/id_rsa`의 텍스트 붙여넣기

![6](https://github.com/user-attachments/assets/c13764e7-1ff0-4c68-bc66-76e1b08470d4)
![7](https://github.com/user-attachments/assets/0d19e216-84e5-43a3-b881-42e5e4f625f2)

---

