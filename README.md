# GithubActionsExample
GithubAction을 공부하고 예제를 적용해보기 위한 repository

# GithubAction

github에서 제공하는 CI(Continuous Integration, 지속 통학) 와 CD(Continuous Deploymeny, 지속 배포) 를 위해 제공하는 서비스. 다른 CI/CD 서비스에 비해 진입장벽이 낮음.

자동으로 코드 저장소에서 어떤 이벤트(event)가 발생했을 때 특정 작업이 일어나게 하거나 주기적으로 어떤 작업들을 반복해서 실행 가능. 

예를 들어, 누군가가 코드 저장소에 Pull Request를 생성하게 되면 GitHub Actions를 통해 해당 코드 변경분에 문제가 없는지 각종 검사, 새로운 코드가 메인(main) 브랜치에 유입(push)되면 GitHub Actions를 통해 소프트웨어를 빌드(build)하고 상용 서버에 배포(deploy)가능. 뿐만 아니라 매일 밤 특정 시각에 그날 하루에 대한 통계 데이터를 수집 가능.

## **Workflows**

GitHub Actions에서 가장 상위 개념인 워크플로우(Workflow, 작업 흐름)는 쉽게 말해 자동화해놓은 작업 과정. 

워크플로우는 코드 저장소 내에서 `.github/workflows` 폴더 아래에 위치한 YAML 파일로 설정하며, 하나의 코드 저장소에는 여러 개의 워크플로우, 즉 여러 개의 YAML 파일을 생성가능.

이 워크플로우 YAML 파일에는 크게 2가지를 정의해야 함. 

1. `on` 속성을 통해서 해당 워크플로우가 언제 실행되는지를 정의.

ex) 코드 저장소의 `main` 브랜치에 `push` 이벤트가 발생할 때 마다 워크플로우를 실행하려면 다음과 같이 설정.

.github/workflows/example.yml

```yaml
on:  
	push:    
		branches:      
			- main

jobs:
  # ...(생략)...
```

다른 예로, 매일 자정에 워크플로우를 실행하려면 다음과 같이 설정.

.github/workflows/hello

```yaml
on:  
	schedule:  
	- cron: "0 0 * * *"

jobs:
  # ...(생략)...
```

1. `jobs` 속성을 통해서 해당 워크플로우가 구체적으로 어떤 일을 해야하는지 명시해야 함

## jobs

GitHub Actions에서 작업(Job)이란 독립된 가상 머신(machine) 또는 컨테이너(container)에서 돌아가는 하나의 처리 단위를 의미. 

하나의 워크플로우는 여러 개의 작업으로 구성되며 적어도 하나의 작업은 있어야 함.

그리고 모든 작업은 기본적으로 동시에 실행되며 필요 시 작업 간에 의존 관계를 설정하여 작업이 실행되는 순서를 제어할 수도 있음.

작업은 워크플로우 YAML 파일 내에서 `jobs` 속성을 사용하며 작업 식별자(ID)와 작업 세부 내용 간의 맵핑(mapping) 형태로 명시.

예를 들어, `job1`, `job2`, `job3`이라는 작업 ID를 가진 3개의 작업을 추가하려면 다음과 같이 설정.

.github/workflows/example.yml

```yaml
# ...(생략)...

jobs:  
	job1:    # job1에 대한 세부 내용  
	job2:    # job2에 대한 세부 내용  
	job3:    # job3에 대한 세부 내용
```

작업의 세부 내용으로는 여러 가지 내용을 명시 가능. 

필수로 들어거야 하는 `runs-on` 속성을 통해 해당 리눅스나 윈도우즈와 같은 실행 환경을 지정.

예를 들어, 가장 널리 사용되는 우분투의 최신 실행 환경에서 해당 작업을 실행하고 싶다면 다음과 같이 설정.

.github/workflows/example.yml

```yaml
# ...(생략)...

jobs:
  job1:
    runs-on: ubuntu-latest    steps:
      # ...(생략)...
```

작업에서 가장 중요한 부분은 작업 순서를 정의하는것. 이 부분은 `steps` 속성을 통해서 설정 가능.

## Steps

GitHub Actions에서는 각 작업(job)이 하나 이상의 단계(step)로 모델링.

작업 단계는 단순한 커맨드(command)나 스크립트(script)가 될 수도 있고 액션(action)이라고 하는 좀 더 복잡한 명령도 가능. 커맨드나 스크립트를 실행할 때는 `run` 속성을 사용하며, 액션을 사용할 때는 `uses` 속성을 사용.

예를 들어 자바스크립트 프로젝트에서 테스트를 돌리려면 코드 저장소에 코드를 작업 실행 환경으로 내려 받고, 패키지를 설치한 후, 테스트 스크립트를 실행. 이 3단계의 작업은 아래와 같이 `steps` 속성을 통해서 명시 가능.

.github/workflows/example.yml

```yaml
# ...(생략)...

jobs:
  test:
    runs-on: ubuntu-latest
    steps:      
			- uses: actions/checkout@v3      
			- run: npm install      
			- run: npm test
```

워크플로우 파일 내에서 작업 단계를 명시해줄 때는 YAML 문법에서 시퀀스(sequence) 타입을 사용하기 때문에 각 단계 앞에 반드시 `-`를 붙여줘야 합니다.

## Actions

액션은 GitHub Actions에서 빈번하게 필요한 반복 단계를 재사용하기 용이하도록 제공되는 일종의 작업 공유 메커니즘. 이 액션은 하나의 코드 저장소 범위 내에서 여러 워크플로우 간에서 공유를 할 수 있을 뿐만 아니라, 공개 코드 저장소를 통해 액션을 공유하면 GitHub 상의 모든 코드 저장소에서 사용이 가능.

GitHub에서 제공하는 대표적인 공개 액션으로 바로 위 예제에서도 사용했던 체크 아웃 액션(`actions/checkout`)을 예로 들면, 대부분의 CI/CD 작업은 코드 저장소로 부터 코드를 작업 실행 환경으로 내려받는 것으로 시작하므로 범용적으로 서용. [GitHub Marketplace](https://github.com/marketplace?type=actions)에서는 수많은 벤더(vendor)가 공개해놓은 다양한 액션을 쉽게 사용 가능.

## 마무리

워크플로우(workflow)는 자동화 시켜놓은 작업 과정을 뜻하며 YAML 파일을 통해 어떤 작업(job)들이 언제 실행되야 하는지를 설정.

각 워크플로우는 독립된 환경에서 실행되는 작업(job)이 적어도 한 개 이상으로 구성되며, 각 작업에는 작업 ID가 부여되고 세부 내용(실행 환경, 작업 단계 등)이 명시.

하나의 작업은 보통 순차적으로 수행되는 여러 개의 단계(step)로 정의되며, 각 단계는 단순한 커맨드(command)일 수도 있고 추상화된 액션(action)일 수도 있음.

# CI/CD With Android

각 YAML의 공통 과정.

1.  pull_request:
    branches: [ develop ]  로 정의된 브랜치로 pull_request가 발생하면, 다음 작업이 실행.
2. job: 을 통해 하나의 작업이 정의(build, deploy). 이 작업은 "ubuntu-latest"에서 실행.
3. "actions/checkout@v3" 액션을 사용하여 코드를 체크아웃. "fetch-depth" 매개변수가 0으로 설정되어 있으므로 이전 히스토리를 가져오지 않음.
4. "Setup JDK 13" 단계에서는 "actions/setup-java@v3" 액션을 사용하여 Zulu를 기반으로 JDK 17을 설치.
5. "Setup Android SDK" 단계에서는 "android-actions/setup-android@v2" 액션을 사용하여 Android SDK를 설정합니다.
6. "Cache Gradle packages" 단계에서는 "actions/cache@v3" 액션을 사용하여 Gradle 패키지를 캐시. 이는 이전 빌드에서 Gradle 의존성을 다시 다운로드하지 않도록 하여 빌드 시간을 단축
7. "Grant execute permission for gradlew" 단계에서는 "chmod" 명령을 사용하여 "gradlew" 스크립트에 실행 권한을 부여.
8. "Decrypt secrets.tar.gpg" 단계에서는 "gpg" 명령을 사용하여 "secrets.tar.gpg" 파일을 복호화. 이 명령은 "SECRET_GPG_PASSWORD" 환경 변수에서 비밀번호를 가져오며, 이 변수는 GitHub Secrets를 통해 제공.
9. "Unzip secrets.tar" 단계에서는 "tar" 명령을 사용하여 "secrets.tar" 파일을 압축 해제.

## CI Example

```yaml
# 워크플로우의 이름 지정.
name: Android CI 

# develop 브랜치에 대해서만 pull request 가 발생했을 때 실행하도록 설정.
on:
  pull_request:
    branches: [ develop ]

# 액션 기본단위인 job 설정.
# Ubuntu latest 버전을 사용하여 실행.
jobs:
  build:
    runs-on: ubuntu-latest

		#step 속성을 통해 작업 순서 정의. 각 작업(job)은 하나 이상의 단계 (step)으로 정의
    # 각 단계 앞에 반드시 - 명시
    #워크플로우가 레포지토리에 접근할 수 있게 checkout
    steps:
      # 워크스페이스를 체크아웃. v3 버전 사용.
      - uses: actions/checkout@v3
        with:
          # 이전 commit 기록들까지 모두 가져오기 위해서 fetch-depth를 0으로 설정.
          fetch-depth: 0

			#안드로이드 프로젝트 빌드를 위한 기본 셋팅
      # JDK 17을 설정.
      - name: Setup JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: "zulu"
          java-version: 17
      
      # 안드로이드 SDK 설정.
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
  
      # Gradle 패키지 캐시.
      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties', '**/buildSrc/**/*.kt') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
      
      # gradlew 파일에 실행 권한 부여.
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # 비밀 정보를 저장한 secrets.tar.gpg 파일을 복호화.
			# 테스트 환경에서는 해당 파일 없음으로 주석
      #- name: Decrypt secrets.tar.gpg
      #  run: gpg --quiet --batch --yes --always-trust --decrypt --passphrase="$SECRET_GPG_PASSWORD" --output secrets.tar secrets.tar.gpg
      #  env:
      #    # GitHub Secrets에서 비밀 정보를 가져와 사용.
      #    SECRET_GPG_PASSWORD: ${{ secrets.SECRET_GPG_PASSWORD }}

      # secrets.tar 파일을 해제.
      #- name: Unzip secrets.tar
      #  run: tar xvf secrets.tar

			# ktlint, detekt 사용하기 위해서는 프로젝트 단위 gradle 에서 해당 라이브러리 import 필요
    
      # ktlint 실행.
      - name: Run ktlint
        run: ./gradlew ktlintCheck
    
      # detekt 실행.
      - name: Run detekt
        run: ./gradlew detekt
    
      # 유닛 테스트 실행.
      - name: Run unit tests
        run: ./gradlew testDebugUnitTest

      # 안드로이드 테스트 보고서를 생성.
			# https://github.com/asadmansr/android-test-report-action
      - name: Create android test report
        uses: asadmansr/android-test-report-action@v1.2.0
        if: ${{ always() }} # 항상 실행.

      # 릴리스 APK.
      - name: Build assemble release apk
        run: ./gradlew assembleRelease
```

1. 문법 검사를 위한 ktlint, detekt 를 실행.
2. Run unit tests 단계에서 testDebugUnitTest 실행.
3. Create android test report 단계에서 안드로이드 테스트 보고서를 생성
4. Build assemble release apk 단계에서 릴리스 APK 빌드.

## CD Example

```yaml
# 워크플로우의 이름
name: Android CD 

# pull_request 이벤트가 발생하고, deploy-dev 브랜치에서 생성된 경우 워크플로우가 실행됨
on:
  pull_request:
    branches: [ deploy-dev ] 

jobs:
  deploy:
    # 작업을 실행할 운영체제
    runs-on: ubuntu-latest 

		#step 속성을 통해 작업 순서 정의. 각 작업(job)은 하나 이상의 단계 (step)으로 정의
    # 각 단계 앞에 반드시 - 명시
    #워크플로우가 레포지토리에 접근할 수 있게 checkout
    steps:
      # 워크스페이스를 체크아웃. v3 버전 사용.
      - uses: actions/checkout@v3
        with:
          # 이전 commit 기록들까지 모두 가져오기 위해서 fetch-depth를 0으로 설정.
          fetch-depth: 0

			#안드로이드 프로젝트 빌드를 위한 기본 셋팅
      # JDK 17을 설정.
      - name: Setup JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: "zulu"
          java-version: 17
      
      # 안드로이드 SDK 설정.
      - name: Setup Android SDK
        uses: android-actions/setup-android@v2
  
      # Gradle 패키지 캐시.
      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties', '**/buildSrc/**/*.kt') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
      
      # gradlew 파일에 실행 권한 부여.
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # 비밀 정보를 저장한 secrets.tar.gpg 파일을 복호화.
			# 테스트 환경에서는 해당 파일 없음으로 주석
      #- name: Decrypt secrets.tar.gpg
      #  run: gpg --quiet --batch --yes --always-trust --decrypt --passphrase="$SECRET_GPG_PASSWORD" --output secrets.tar secrets.tar.gpg
      #  env:
      #    # GitHub Secrets에서 비밀 정보를 가져와 사용.
      #    SECRET_GPG_PASSWORD: ${{ secrets.SECRET_GPG_PASSWORD }}

      # secrets.tar 파일을 해제.
      #- name: Unzip secrets.tar
      #  run: tar xvf secrets.tar

      # 릴리스 유니버설 APK 빌드
      - name: Build release universal apk
        run: ./gradlew presentation:packageReleaseUniversalApk

      # Firebase App Distribution에 APK 업로드
      - name: Upload apk to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }} # Firebase App ID
          token: ${{ secrets.FIREBASE_TOKEN }} # Firebase 토큰
          groups: runnerbe # 배포 대상 그룹
          file: presentation/build/outputs/universal_apk/release/presentation-release-universal.apk # 업로드 할 APK 파일 경로
          releaseNotesFile: documents/release-note/default-note.txt # 릴리스 노트 파일 경로
```

1. "Build release universal apk" 단계에서는 "gradlew" 스크립트를 사용하여 "presentation:packageReleaseUniversalApk" 작업을 실행하여 APK 파일을 빌드.
2.  "Upload apk to Firebase App Distribution" 단계에서는 "wzieba/Firebase-Distribution-Github-Action@v1" 액션을 사용하여 Firebase App Distribution에 APK 파일을 업로드. 이를 위해 "FIREBASE_APP_ID" 및 "FIREBASE_TOKEN" GitHub Secrets를 사용. 이 작업은 "presentation/build/outputs/universal_apk/release/presentation-release-universal.apk" 파일을 업로드, 릴리스 노트는 "documents/release-note/default-note.txt"에서 가져옴.
