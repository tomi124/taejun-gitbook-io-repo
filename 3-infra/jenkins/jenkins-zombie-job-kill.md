# Jenkins zombie job kill

![](<../../.gitbook/assets/스크린샷 2022-03-11 오후 1.01.04.png>)

```
Jenkins.instance.getItemByFullName("젠킨스 잡 명")
                .getBuildByNumber(12450) // job build number
                .finish(
                        hudson.model.Result.ABORTED,
                        new java.io.IOException("Aborting build")
                );
```
