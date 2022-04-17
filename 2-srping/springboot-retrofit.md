# SpringBoot와 Retrofit 연동

#### **\[ SpringBoot와 Retrofit 연동 ]**

SpringBoot 프로젝트에서 Retrofit2를 사용하기 위해서는 2가지 의존성을 추가해주어야 한다.

```
// https://mvnrepository.com/artifact/com.squareup.retrofit2/retrofit
compile group: 'com.squareup.retrofit2', name: 'retrofit', version: '2.9.0'

// https://mvnrepository.com/artifact/com.squareup.retrofit2/converter-gson
compile group: 'com.squareup.retrofit2', name: 'converter-gson', version: '2.9.0'
```

각각은 Retrofit을 구현하기 위한 의존성과 Response를 변환하기 위한 Converter를 위한 의존성이다.

그리고 호출할 API를 명세하는 인터페이스를 구현해야 하는데, 우선은 Retrofit2의 사용법을 모른다는 가정 하에 빈 인터페이스를 생성해두도록 하자. 이름이 명시적이면 좋겠지만, 예시 코드를 작성중이므로 ServerAPIs라고 해두겠다. 이후에 서버로 호출할 API는 이 인터페이스에 구현을 하면 된다.

```
public interface ServerAPIs {
    
}
```

&#x20;

의존성을 추가하고 빌드를 하면 Retrofit 관련 설정 클래스를 추가할 수 있다.

```
@Configuration
public class RetrofitConfig {

    private static final String BASE_URL = "http://localhost:8080/";

    @Bean
    public OkHttpClient okHttpClient() {
        return new OkHttpClient.Builder().connectTimeout(20, TimeUnit.SECONDS)
                .writeTimeout(60, TimeUnit.SECONDS)
                .readTimeout(60, TimeUnit.SECONDS)
                .build();
    }

    @Bean
    public Retrofit retrofit(OkHttpClient client) {
        String baseURL = BASE_URL + "api/";
        return new Retrofit.Builder().baseUrl(baseURL)
                .addConverterFactory(GsonConverterFactory.create())
                .client(client)
                .build();
    }

    @Bean
    public ServerAPIs serverAPIs(Retrofit retrofit) {
        return retrofit.create(ServerAPIs.class);
    }
}
```

&#x20;

RetrofitConfig 클래스에서는 bean을 생성하고 있으므로 @Configuration 어노테이션을 붙여준다. Retrofit은 내부적으로 OkHttpClient를 사용하고 있으므로 해당 Bean을 구현하도록 하자. 각각의 타임아웃 시간이나 URL 등은 자신의 상황에 맞게 변형하면 된다. 그리고 해당 Client를 기반으로 Retrofit 클래스를 Bean으로 생성해주면 되는데, 여기서 주의해야 할 것이 하나 있다. Retrofit 클래스를 생성할 때 baseUrl의 끝이 항상 "/"로 끝나도록 해야 한다는 것이다. 위의 예제에서는 api/ 로 끝나고 있으므로 해당 조건을 만족하고 있다. 또한 서버가 json 형태로 데이터를 반환한다는 가정 하에 GsonConverterFactory.create()를 사용한 것이므로, 다른 형태로 데이터가 반환될 때에는 이 부분을 수정하면 된다.

이렇게 설정을 해주면 이제 API를 호출할 수 있는 준비가 되었다. 실제로 호출할 API 정보는 ServerAPIs 인터페이스에 구현을 하면 되는데, Retrofit2 사용법에 대해 자세히 알아보도록 하자.

&#x20;

&#x20;

### **2. Retrofit2 사용법**

***

#### **\[ Retrofit2 어노테이션 ]**

* @GET, @POST, @PUT, @DELETE: HTTP 메소드를 지정한다.
* @Path: 동적으로 URL을 바인딩한다.
* @Header: HTTP 헤더에 해당 값을 추가한다.
* @Field: Post 방식에서만 사용가능하며, form-urlencoded 형태로 데이터를 전송한다.
* @Query: 쿼리 스트링 파라미터(Query String Parameter)를 추가한다.
* @Body: Post 방식에서만 사용가능하며, json 형태로 데이터를 전송한다
* @URL: 동적인 URL이 필요할 때 사용한다.
* @Multipart: Multipart/form-data 형태로 Multipart 요청을 보내기 위해 사용한다.

&#x20;

#### **\[ Retrofit2 사용 예제 ]**

[네이버 웍스의 API 문서](https://developers.worksmobile.com/kr/document/2?lang=ko)를 일부 참고하여 해당 내용을 기준으로 인터페이스를 완성해보도록 하자.

(하위의 코드들을 이해를 돕기 위해 원래의 코드들로부터 짜집기되었다.)

&#x20;

&#x20;

**1. 파일 다운로드 API**

문서 링크: [developers.worksmobile.com/kr/document/1008038?lang=ko](https://developers.worksmobile.com/kr/document/1008038?lang=ko)

&#x20;

파일 다운로드 API는 GET 방식을 지원하고 있으며, 총 3개의 Header와 2개의 Path 변수를 파라미터로 받고 있다. 그 중에서 consumerKey는 모든 API에 대해 항상 동일한 Header인데, 모든 API에 넣어주면 번거로울 수 있다. 그러므로 우선 consumerKey는 OkHttpClient의 Interceptor에서 처리하도록 하자.

```
@Bean
public OkHttpClient okHttpClient() {
    return new OkHttpClient.Builder().connectTimeout(20, TimeUnit.SECONDS)
            .writeTimeout(60, TimeUnit.SECONDS)
            .readTimeout(60, TimeUnit.SECONDS)
            // 인터셉터 추가
            .addInterceptor(chain -> {
                Request request = chain.request().newBuilder().addHeader("consumerKey", "컨슈머키").build();
                return chain.proceed(request);
            })
            .build();
}
```

위와 같이 OkHttpClient에 인터셉터를 추가함으로써 HTTP 요청이 전달되기 전에 항상 consumerKey가 헤더에 등록되도록 공통 부분을 빼낼 수 있다.

&#x20;

그리고 이제 API 명세에 따라 2가지의 Header와 2가지 Path를 추가해주면 다음과 같이 작성할 수 있다.

```
public interface ServerAPIs {
    
	@GET("{resourceLocation}/v2/files/{resourceKey}/download")
	Call<ResponseBody> downloadFile(
		@Header("X-DRIVE-API-TYPE") String apiType,
		@Header("Authorization") String token,
		@Path("resourceLocation") String resourceLocation,
		@Path("resourceKey") String resourceKey
	);
    
}
```

Retrofit의 반환값은 Call\<T>의 형태이다. 파일 다운로드 API는 서버로부터 받는 데이터가 byte\[]이므로, 반환형을 okhttp3의 Responsebody로 명시를 해주면 된다.

위와 같이 작성한 API를 실제로 호출하여 결과를 받는 로직은 다음과 같다.

```
@Override
public byte[] downloadFile(String token, String fileName, String parentKey, String resourceKey) {
	Call<ResponseBody> call = serverAPIs.downloadFile("reseller-api", token, parentKey, resourceKey);
	try {
		Response<ResponseBody> response = call.execute();
		if (response.isSuccessful()) {
			return response.body().bytes();
		}
		log.error(response.errorBody().string());
	} catch (IOException e) {
		log.error(e.getMessage());
	}
	return new byte[] {};
}
```

먼저 serverAPIs 빈을 통해 downladFile 함수를 호출하고 있고, 그에 맞는 파라미터를 넘겨주고 있다.

그러면 Call을 반환값으로 받게 되는데, 해당 Call을 동기적으로 호출하기 위해 call.execute()를 사용하였다. 그러변 반환값으로 Response\<T>를 받게 되고, isSuccessful()을 통해 API가 성공적으로 호출되었는지 확인할 수 있다.

&#x20;

&#x20;

**2. 파일 업로드 API**

문서 링크: [developers.worksmobile.com/kr/document/1008037?lang=ko](https://developers.worksmobile.com/kr/document/1008037?lang=ko)

&#x20;

파일 업로드 API는 POST 방식을 지원하고 있으며, 총 1개의 Header와 1개의 Path 변수를 파라미터로 받고 있다. 또한 6개의 Form 파라미터를 보내고, 그 중에서 실제 파일 데이터는 마지막에 위채해야 한다고 명세되어 있다. 그렇기 때문에 이러한 명세에 따라 다음과 같이 인터페이스를 작성하였다.

```
public interface ServerAPIs {
    
	@Multipart
	@POST("{resourceLocation}/v2/files")
	Call<FileUploadResponse> uploadFile(
		@Header("X-DRIVE-API-TYPE") String apiType,
		@Header("Authorization") String token,
		@Path("resourceLocation") Integer resourceLocation,
		@PartMap Map<String, RequestBody> map,
		@Part MultipartBody.Part file
	);
    
}
```

먼저 파일을 포함한 Multipart 요청이기 때문에 @Multipart 어노테이션이 추가되었고, @POST 방식으로 작성되었다. 또한 각각의 필요한 header와 Path가 추가되었으며, 파일을 제외한 Form 파라미터를 담는 Map과 파일 자체를 담는 MultipartBody.Part가 추가되었다. MultipartBody.Part에는 반드시 @Part 어노테이션이 붙어야하며, Form 파라미터를 담는 Map에는 @PartMap이 추가되어야 한다. 또한 Call의 타입은 문서에 따라 다음과 같이 클래스를 추가해주면 된다.

```
@Getter
@NoArgsConstructor(force = true)
public class FileUploadResponse implements Serializable {

	private final String versionNo;
	private final String resourceKey;
	private final String shareNo;
	private final String resourceNo;

}
```

&#x20;

이를 실제로 호출하는 코드는 다음과 같다.

```
@Override
public void uploadFile(String token, String fileName, String parentKey) {
	Call<FileUploadResponse> call = naverFileAPIs.uploadFile("reseller-api", token,
		resourceLocation, createPartMap(fileName, parentKey), createMultipartBodyPart(fileName));
	sendFileUploadAPI(fileName, call);
}

private void sendFileUploadAPI(String fileName, Call<FileUploadResponse> call) {
	call.enqueue(new Callback<FileUploadResponse>() {
		@Override
		public void onResponse(Call<FileUploadResponse> call, Response<FileUploadResponse> response) {
            if (!response.isSuccessful()) {
				try {
					log.error(response.errorBody().string());
				} catch (IOException e) {
					log.error(e.getMessage());
				}
			}
		}
		@Override
		public void onFailure(Call<FileUploadResponse> call, Throwable t) {
			log.error(t.getMessage());
		}
	});
}

private Map<String, RequestBody> createPartMap(String fileName, String toParentKey) {
	Map<String, RequestBody> map = new HashMap<>();
	map.put("toParentKey", createMultipartFormParameter(toParentKey));
	map.put("resourceName", createMultipartFormParameter(fileName));
	return map;
}

private MultipartBody.Part createMultipartBodyPart(String fileName) {
	File file = new File(fileName);
	RequestBody requestFile = RequestBody.create(MediaType.parse("multipart/form-data"), file);
	return MultipartBody.Part.createFormData("FileData", fileName, requestFile);
}

private RequestBody createMultipartFormParameter(String content) {
	return RequestBody.create(MediaType.parse("text/plain"), content);
}
```

우선 MultiPartForm 형태의 파라미터를 생성하기 위한 createMultipartFormParameter 함수와 이를 통해 PartMap을 만들기 위한createPartMap함수를 작성하였다. 그리고 Part를 만들기 위한 createMultipartBodyPart 함수를 추가하였다. 앞선 예제와 마찬가지로 Call\<T>를 생성하였는데, 위의 예제와는 다르게 enqueue를 통해 비동기 호출을 하고 있다. enqueue의 파라미터로는 CallBack을 넣어주어야 하는데, CallBack은 2가지 함수를 오버라이드 해주어야 한다. 각각은 onResponse 함수와 onFailure 함수이다. onFailure는 서버와 통신 중 네트워크 예외가 발생하거나 요청을 처리하는 중 예기치 않은 예외가 발생했을때 호출된다. 그 외에 상황은 onSuccess가 호출되는데, 마찬가지로 isSuccessful()함수를 통해 API의 성공 또는 실패 여부를 확인할 수 있다.&#x20;

&#x20;

&#x20;

**3. 파일 목록 조회 API**

문서 링크: [developers.worksmobile.com/kr/document/1008006?lang=ko](https://developers.worksmobile.com/kr/document/1008006?lang=ko)

&#x20;

```
public interface ServerAPIs {
    
	@GET("drive/rl/{resourceLocation}/v2/files/{resourceKey}/children")
	Call<FileResponse> getFileList(
		@Header("Authorization") String token,
		@Path("resourceLocation") Integer resourceLocation,
		@Path("resourceKey") String resourceKey,
		@Query("offset") Integer offset,
		@Query("limit") Integer limit
	);
    
}
```

&#x20;

```
@Getter
public class FileResponse implements Serializable {

	private Integer offset;
	private Boolean hasMore;
	private List<FileItem> items;

	public FileResponse() {
		this.offset = 0;
		this.hasMore = false;
		this.items = new ArrayList<>();
	}

}
```

&#x20;

```
public FileResponse getFileList(String token, String resourceKey, Integer offset, Integer limit) {
	Call<FileResponse> call = naverServiceAPIs.getFileList(token, 0, resourceKey, offset, limit);
	try {
		Response<FileResponse> response = call.execute();
		if (response.isSuccessful()) {
			return response.body();
		}
		log.error(response.errorBody().string());
	} catch (IOException e) {
		log.error(e.getMessage());
	}
	return new FileResponse();
}
```

\
\
출처: [https://mangkyu.tistory.com/124](https://mangkyu.tistory.com/124) \[MangKyu's Diary]
