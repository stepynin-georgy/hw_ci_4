# Домашнее задание к занятию 11 «Teamcity»

## Подготовка к выполнению

1. В Yandex Cloud создайте новый инстанс (4CPU4RAM) на основе образа `jetbrains/teamcity-server`.
2. Дождитесь запуска teamcity, выполните первоначальную настройку.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_44.png)

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_45.png)

3. Создайте ещё один инстанс (2CPU4RAM) на основе образа `jetbrains/teamcity-agent`. Пропишите к нему переменную окружения `SERVER_URL: "http://<teamcity_url>:8111"`.
4. Авторизуйте агент.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_47.png)

5. Сделайте fork [репозитория](https://github.com/aragastmatb/example-teamcity).

   [Fork репозитория](https://github.com/stepynin-georgy/example-teamcity)

6. Создайте VM (2CPU4RAM) и запустите [playbook](./infrastructure).

<details>
<summary>Запуск playbook</summary>

```
(venv_ci) root@ansible-ubuntu:/opt/hw_ci_4# ansible-playbook -i infrastructure/inventory/cicd/hosts.yml infrastructure/site.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.15 (default, Sep 13 2024, 08:34:12) [GCC 13.2.0]. This
 feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
/opt/venv_ci/lib/python3.6/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 3.6 is no longer supported by the Python core team. Therefore, support for it is deprecated in cryptography. The next release of cryptography will remove support for Python 3.6.
  from cryptography.exceptions import InvalidSignature

PLAY [Get Nexus installed] ***************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************
The authenticity of host '130.193.53.249 (130.193.53.249)' can't be established.
ED25519 key fingerprint is SHA256:xpzqRjp1/9qFQ4OChC5Fz4oN4tAJqQX962Libt7VSVU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [nexus-01]

TASK [Create Nexus group] ****************************************************************************************************************************************************************
changed: [nexus-01]

TASK [Create Nexus user] *****************************************************************************************************************************************************************
changed: [nexus-01]

TASK [Install JDK] ***********************************************************************************************************************************************************************
changed: [nexus-01]

TASK [Create Nexus directories] **********************************************************************************************************************************************************
changed: [nexus-01] => (item=/home/nexus/log)
changed: [nexus-01] => (item=/home/nexus/sonatype-work/nexus3)
changed: [nexus-01] => (item=/home/nexus/sonatype-work/nexus3/etc)
changed: [nexus-01] => (item=/home/nexus/pkg)
changed: [nexus-01] => (item=/home/nexus/tmp)

TASK [Download Nexus] ********************************************************************************************************************************************************************
[WARNING]: Module remote_tmp /home/nexus/.ansible/tmp did not exist and was created with a mode of 0700, this may cause issues when running as another user. To avoid this, create the
remote_tmp dir with the correct permissions manually
changed: [nexus-01]

TASK [Unpack Nexus] **********************************************************************************************************************************************************************
changed: [nexus-01]

TASK [Link to Nexus Directory] ***********************************************************************************************************************************************************
changed: [nexus-01]

TASK [Add NEXUS_HOME for Nexus user] *****************************************************************************************************************************************************
changed: [nexus-01]

TASK [Add run_as_user to Nexus.rc] *******************************************************************************************************************************************************
changed: [nexus-01]

TASK [Raise nofile limit for Nexus user] *************************************************************************************************************************************************
changed: [nexus-01]

TASK [Create Nexus service for SystemD] **************************************************************************************************************************************************
changed: [nexus-01]

TASK [Ensure Nexus service is enabled for SystemD] ***************************************************************************************************************************************
changed: [nexus-01]

TASK [Create Nexus vmoptions] ************************************************************************************************************************************************************
changed: [nexus-01]

TASK [Create Nexus properties] ***********************************************************************************************************************************************************
changed: [nexus-01]

TASK [Lower Nexus disk space threshold] **************************************************************************************************************************************************
skipping: [nexus-01]

TASK [Start Nexus service if enabled] ****************************************************************************************************************************************************
changed: [nexus-01]

TASK [Ensure Nexus service is restarted] *************************************************************************************************************************************************
skipping: [nexus-01]

TASK [Wait for Nexus port if started] ****************************************************************************************************************************************************
ok: [nexus-01]

PLAY RECAP *******************************************************************************************************************************************************************************
nexus-01                   : ok=17   changed=15   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

/opt/venv_ci/bin/ansible-playbook:135: ResourceWarning: unclosed file <_io.BufferedRandom name=5>
  exit_code = cli.run()
(venv_ci) root@ansible-ubuntu:/opt/hw_ci_4#

```

</details>

## Основная часть

1. Создайте новый проект в teamcity на основе fork.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_46.png)

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_48.png)

2. Сделайте autodetect конфигурации.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_51.png)

3. Сохраните необходимые шаги, запустите первую сборку master.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_52.png)

4. Поменяйте условия сборки: если сборка по ветке `master`, то должен происходит `mvn clean deploy`, иначе `mvn clean test`.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_51.png)

5. Для deploy будет необходимо загрузить [settings.xml](./teamcity/settings.xml) в набор конфигураций maven у teamcity, предварительно записав туда креды для подключения к nexus.

clean deploy ```teamcity.build.branch contains master```

clean test ```teamcity.build.branch does not contain master```

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_53.png)

6. В pom.xml необходимо поменять ссылки на репозиторий и nexus.

[pom.xml](https://github.com/stepynin-georgy/example-teamcity/blob/master/pom.xml)

7. Запустите сборку по master, убедитесь, что всё прошло успешно и артефакт появился в nexus.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_55.png)

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_56.png)

8. Мигрируйте `build configuration` в репозиторий.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_57.png)

``` TeamCity change in 'netology' project: Versioned settings configuration updated, collecting changed files... ```

9. Создайте отдельную ветку `feature/add_reply` в репозитории.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_59.png)

10. Напишите новый метод для класса Welcomer: метод должен возвращать произвольную реплику, содержащую слово `hunter`.

[HelloPlayer.java](https://github.com/stepynin-georgy/example-teamcity/blob/master/src/main/java/plaindoll/HelloPlayer.java)

[Welcomer.java](https://github.com/stepynin-georgy/example-teamcity/blob/master/src/main/java/plaindoll/Welcomer.java)

11. Дополните тест для нового метода на поиск слова `hunter` в новой реплике.

[WelcomerTest.java](https://github.com/stepynin-georgy/example-teamcity/blob/master/src/test/java/plaindoll/WelcomerTest.java)

12. Сделайте push всех изменений в новую ветку репозитория.
13. Убедитесь, что сборка самостоятельно запустилась, тесты прошли успешно.

```
10:43:40 Finalize build settings
10:43:40 The build is removed from the queue to be prepared for the start
10:43:41 Collecting changes in 1 VCS root
10:43:41 Starting the build on the agent "ip_62.84.120.214"
10:43:41 Updating tools for build
10:43:41 Clearing temporary directory: /opt/buildagent/temp/buildTmp
10:43:41 Publishing internal artifacts
10:43:42 Using vcs information from agent file: 19c9ba3732cfdc1c.xml
10:43:42 Checkout directory: /opt/buildagent/work/19c9ba3732cfdc1c
10:43:42 Updating sources: auto checkout (on agent)
10:43:43 Step 1/2: clean deploy (Maven)
10:43:43 Step 2/2: clean test (Maven)
10:43:43   Using predefined Maven user settings: settings.xml
10:43:43   Using predefined Maven user settings: settings.xml
10:43:43   Initial M2_HOME not set
10:43:43   Current M2_HOME = /opt/buildagent/tools/maven3_9
10:43:43   PATH = /opt/buildagent/tools/maven3_9/bin:/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
10:43:43   Using watcher: /opt/buildagent/plugins/mavenPlugin/maven-watcher-jdk17/maven-watcher-agent.jar
10:43:43   Using agent local repository at /opt/buildagent/system/jetbrains.maven.runner/maven.repo.local
10:43:43   *** Start reading the project structure ***
10:43:46   Initial MAVEN_OPTS not set
10:43:46   Current MAVEN_OPTS not set
10:43:46   Starting: /opt/java/openjdk/bin/java -Dagent.home.dir=/opt/buildagent -Dagent.name=ip_62.84.120.214 -Dagent.ownPort=9090 -Dagent.work.dir=/opt/buildagent/work -Dbuild.number=6 -Dbuild.vcs.number=2e5f2b98565c95380177f7f7968994640881d5f1 -Dbuild.vcs.number.1=2e5f2b98565c95380177f7f7968994640881d5f1 -Dbuild.vcs.number.Netology_HttpsGithubComStepyninGeorgyExampleTeamcityGitRefsHeadsMaster=2e5f2b98565c95380177f7f7968994640881d5f1 -Dclassworlds.conf=/opt/buildagent/temp/buildTmp/teamcity.m2.conf -Dcom.jetbrains.maven.watcher.report.file=/opt/buildagent/temp/buildTmp/maven-build-info.xml -Djava.io.tmpdir=/opt/buildagent/temp/buildTmp -Dmaven.home=/opt/buildagent/tools/maven3_9 -Dmaven.multiModuleProjectDirectory=/opt/buildagent/work/19c9ba3732cfdc1c -Dteamcity.agent.cpuBenchmark=450 -Dteamcity.agent.dotnet.agent_url=http://localhost:9090/RPC2 -Dteamcity.agent.dotnet.build_id=6 -Dteamcity.auth.password=******* -Dteamcity.auth.userId=TeamCityBuildId=6 -Dteamcity.build.changedFiles.file=/opt/buildagent/temp/buildTmp/teamcity.changedFiles.txt -Dteamcity.build.checkoutDir=/opt/buildagent/work/19c9ba3732cfdc1c -Dteamcity.build.id=6 -Dteamcity.build.properties.file=/opt/buildagent/temp/buildTmp/teamcity.build.parameters -Dteamcity.build.properties.file.checksum=/opt/buildagent/temp/buildTmp/teamcity.build.parameters.checksum -Dteamcity.build.tempDir=/opt/buildagent/temp/buildTmp -Dteamcity.build.workingDir=/opt/buildagent/work/19c9ba3732cfdc1c -Dteamcity.buildConfName=Build -Dteamcity.buildType.id=Netology_Build -Dteamcity.configuration.properties.file=/opt/buildagent/temp/buildTmp/teamcity.config.parameters -Dteamcity.maven.userSettings.path=/opt/buildagent/temp/buildTmp/maven_settings_17548681971448702347.xml -Dteamcity.maven.watcher.home=/opt/buildagent/plugins/mavenPlugin/maven-watcher-jdk17 -Dteamcity.projectName=netology -Dteamcity.runner.properties.file=/opt/buildagent/temp/buildTmp/teamcity.runner.parameters -Dteamcity.tests.recentlyFailedTests.file=/opt/buildagent/temp/buildTmp/teamcity.testsToRunFirst.txt -Dteamcity.version=2024.07.2 (build 160695) -Dmaven.repo.local=/opt/buildagent/system/jetbrains.maven.runner/maven.repo.local -classpath /opt/buildagent/tools/maven3_9/boot/plexus-classworlds-2.7.0.jar: org.codehaus.plexus.classworlds.launcher.Launcher -f /opt/buildagent/work/19c9ba3732cfdc1c/pom.xml -B -s /opt/buildagent/temp/buildTmp/maven_settings_17548681971448702347.xml -Dmaven.test.failure.ignore=true clean test
10:43:46   in directory: /opt/buildagent/work/19c9ba3732cfdc1c
10:43:47   [INFO] Scanning for projects...
10:43:48   [INFO]
10:43:48   [INFO] -----------------------< org.netology:plaindoll >-----------------------
10:43:48   [INFO] Building plaindoll 0.0.2
10:43:48   [INFO]   from pom.xml
10:43:48   [INFO] --------------------------------[ jar ]---------------------------------
10:43:48   org.netology:plaindoll
10:43:48   Importing data from '/opt/buildagent/work/19c9ba3732cfdc1c/target/failsafe-reports/TEST-*.xml' with 'surefire' processor
10:43:48   Importing data from '/opt/buildagent/work/19c9ba3732cfdc1c/target/surefire-reports/TEST-*.xml' with 'surefire' processor
10:43:48   Surefire report watcher
10:43:48   Surefire report watcher
10:43:48   [INFO]
10:43:48   [INFO] --- clean:3.2.0:clean (default-clean) @ plaindoll ---
10:43:48   [INFO] Deleting /opt/buildagent/work/19c9ba3732cfdc1c/target
10:43:48   [INFO]
10:43:48   [INFO] --- resources:3.3.1:resources (default-resources) @ plaindoll ---
10:43:48   [WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
10:43:48   [INFO] skip non existing resourceDirectory /opt/buildagent/work/19c9ba3732cfdc1c/src/main/resources
10:43:48   [INFO]
10:43:48   [INFO] --- compiler:3.11.0:compile (default-compile) @ plaindoll ---
10:43:48   [INFO] Changes detected - recompiling the module! :source
10:43:48   [WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
10:43:48   [INFO] Compiling 2 source files with javac [debug target 1.8] to target/classes
10:43:49   [WARNING] bootstrap class path not set in conjunction with -source 8
10:43:49   [INFO]
10:43:49   [INFO] --- resources:3.3.1:testResources (default-testResources) @ plaindoll ---
10:43:49   [WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
10:43:49   [INFO] skip non existing resourceDirectory /opt/buildagent/work/19c9ba3732cfdc1c/src/test/resources
10:43:49   [INFO]
10:43:49   [INFO] --- compiler:3.11.0:testCompile (default-testCompile) @ plaindoll ---
10:43:49   [INFO] Changes detected - recompiling the module! :dependency
10:43:49   [WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
10:43:49   [INFO] Compiling 1 source file with javac [debug target 1.8] to target/test-classes
10:43:49   [WARNING] bootstrap class path not set in conjunction with -source 8
10:43:49   [INFO]
10:43:49   [INFO] --- surefire:3.2.2:test (default-test) @ plaindoll ---
10:43:50   [INFO] Using auto detected provider org.apache.maven.surefire.junit4.JUnit4Provider
10:43:50   [INFO]
10:43:50   [INFO] -------------------------------------------------------
10:43:50   [INFO]  T E S T S
10:43:50   [INFO] -------------------------------------------------------
10:43:50   [INFO] Running plaindoll.WelcomerTest
10:43:50   [INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.089 s -- in plaindoll.WelcomerTest
10:43:50   [INFO]
10:43:50   [INFO] Results:
10:43:50   [INFO]
10:43:50   [INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
10:43:50   [INFO]
10:43:50   [INFO] ------------------------------------------------------------------------
10:43:50   [INFO] BUILD SUCCESS
10:43:50   [INFO] ------------------------------------------------------------------------
10:43:50   [INFO] Total time:  2.884 s
10:43:50   [INFO] Finished at: 2024-09-20T11:43:50+01:00
10:43:50   [INFO] ------------------------------------------------------------------------
10:43:50   Process exited with code 0
10:43:50   Publishing artifacts
10:43:51   Waiting for 2 service processes to complete
10:43:51   Surefire report watcher
10:43:51 Publishing artifacts
10:43:51 Publishing internal artifacts
10:43:52 Publishing internal artifacts
10:43:52 Build finished
```

14. Внесите изменения из произвольной ветки `feature/add_reply` в `master` через `Merge`.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_64.png)

16. Убедитесь, что нет собранного артефакта в сборке по ветке `master`.
17. Настройте конфигурацию так, чтобы она собирала `.jar` в артефакты сборки.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_65.png)

18. Проведите повторную сборку мастера, убедитесь, что сбора прошла успешно и артефакты собраны.

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_66.png)

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_67.png)

![изображение](https://github.com/stepynin-georgy/hw_ci_4/blob/main/img/Screenshot_68.png)

19. Проверьте, что конфигурация в репозитории содержит все настройки конфигурации из teamcity.
20. В ответе пришлите ссылку на репозиторий.

[Репозиторий](https://github.com/stepynin-georgy/example-teamcity)

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
