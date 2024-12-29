### 로컬 리포지토리로 업로드 하기
프로젝트들의 공통코드만 따로 빼서 상위 프로젝트로 만들었다고 할때 로컬에 리포지토리를 잡고 업로드후, 이를 하위 프로젝트에서 디펜던시를 걸어줄수 있음.

상위 프로젝트 gradle
```gradle
uploadArchives {
      repositories {
          mavenDeployer {
              repository(url: 'file://' + new File(System.getProperty('user.home'), 'LocalRepository').absolutePath)

              pom.project {
                  developers {
                      developer {
                          id 'cchcc'
                          name 'Jo Hyun chul'
                          email 'jo.cchcc@gmail.com'
                      }
                  }
              }

          }
      }
  }
```
업로드
```
./gradlew uploadArchives
```

하위 프로젝트 gradle
```gradle
repositories {
    maven {url 'file://' + new File(System.getProperty('user.home'), 'LocalRepository').absolutePath}
}
dependencies {
    compile 'my-project:common-module:0.1'  // 상위 프로젝트
}
```
<http://turhanoz.com/gradle-dependency-and-local-repository/>
  
### vertx fatjar 만들기
```gradle
task fatJar(type: Jar) {
    manifest {
        attributes 'Manifest-Version': version,
                'Main-Class': 'io.vertx.core.Launcher',
                'Main-Verticle': 'cchcc.MainVerticle'   // entry verticle
    }
    baseName = project.name + '-fat-runnable'
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
    exclude ("**/config.json")  // 뺄거
    with jar
}
```
