# 그림으로 보는 Filter와 Interceptor

### 서블릿 필터 vs 스프링 인터셉터

서블릿 필터와 스프링 인터셉터는 모두 웹과 관련된 공통 관심사를 처리하는데 사용되는 기술이다.필터는 서블릿에서 제공하고 인터셉터는 스프링 MVC가 제공하는 기술인데, 적용되는 순서와 범위, 그리고 사용방법이 다르다.

### **필터와 인터셉터의 흐름**

****

![](https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F56ce423c-52ed-405e-affa-3099ed41b373%2FUntitled.png\&blockId=9f459275-0c70-4c00-9b4b-d01e3eb8556d)

•필터를 적용하면 필터가 호출된 이후 서블릿이 호출된다. (여기서 서블릿은 스프링의 경우 디스패처 서블릿을 의미한다고 생각하면 된다.)

•인터셉터를 적용하면 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출된다.

•필터는 서블릿 호출전에 인터셉터는 서블릿 호출 이후 호출되기에 인터셉터는 서블릿에서 예외가 발생한다면 호출되지 않는다.



### **필터와 인터셉터의 제한**

****

![](https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F8da58bd3-99b8-4f15-bc94-204e8974ce1b%2FUntitled.png\&blockId=2af3f62f-2ee6-40c5-becc-43a1ea7ec905)

•필터와 인터셉터는 각각 요청이 적절하지 않을경우 자신의 상태에서 종료할 수 있다.

•필터는 서블릿까지도 가지 못하지만, 스프링 인터셉터는 서블릿까진 통과 후 제한이 된다.

### **필터와 인터셉터 체인**

****

![](https://oopy.lazyrockets.com/api/v2/notion/image?src=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F9826c991-9ed2-4914-bd81-878f7db59b62%2FUntitled.png\&blockId=bf1761a1-8f0a-4219-9c4f-3b2671351e8d)

•둘 다 자유롭게 필터 및 인터셉터를 추가할 수 있다.

•로그를 남기는 필터(혹은 인터셉터)를 적용 후 그 다음 로그인 여부를 체크하는 필터(혹은 인터셉터)를 만들어 적용할 수 있다.
