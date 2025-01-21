---
title: SpringBoot에서 Azure Blob Storage로 사진을 저장하기
date: 2025-1-21 20:25:00 +/-TTTT
categories: [Project, COASTER]
tags: [azure, blob-storage, infra, springboot]
math: true
---

# 사진 저장?

Azure 에서는 Azure Blob Storage를 통해 미디어 파일의 간단한 저장을 지원합니다. 

COASTER 프로젝트 역시도 유저의 프로필 이미지, 게시물 이미지 등 다양한 곳에서 이미지 저징이 필요하기에, Blob Storage를 사용하게 되었습니다. 다만, AWS의 S3와 다르게 블로그 등에서 정리한 문서가 많지 않아 정리해보도록 하겠습니다.

# Azure Portal

## Storage 계정 설정

![img](/assets/img/2025-01-21-azure-blob-storage/0.png){: w="600" }
_storage 계정 설정_

저는 Visual Studio Enterprise 구독을 받고 있어, 해당 구독으로 설정해주었습니다.

지역은 `korea central`로 설정해주었으며, 적은 요금을 위해 성능은 표준으로, 중복도는 기본 지역에서 데이터를 동기적으로 3번 복사하는 LRS를 선택해주었습니다. 중복도 옵션에 대한 자세한 설명은 [여기](https://learn.microsoft.com/ko-kr/azure/storage/common/storage-redundancy#redundancy-in-the-primary-region?wt.mc_id=studentamb_296881)에서 확인하실 수 있습니다.


## 컨테이너 설정

Storage 계정을 설정한 뒤, 스토리지 계정 > 스토리지 브라우저 > Blob 컨테이너에서 새로운 컨테이너를 추가할 수 있습니다.

![img](/assets/img/2025-01-21-azure-blob-storage/1.png){: w="700" }
_스토리지 브라우저 접속_

<br>


저는 이미지를 저장하기 위해 컨테이너 이름을 `coaster-image`라고 작성하여 만들어주었습니다. 또한, 클라이언트에서 url을 통한 이미지 접근이 가능하도록 익명 액세스 수준을 컨테이너로 풀어주었습니다.

![img](/assets/img/2025-01-21-azure-blob-storage/2.png){: w="300" }
_container 설정_

# Spring

> 공식 문서: [Java용 Azure Blob Storage 클라이언트 라이브러리](https://learn.microsoft.com/ko-kr/azure/storage/blobs/storage-quickstart-blobs-java?tabs=powershell%2Cmanaged-identity%2Croles-azure-portal%2Csign-in-azure-cli&pivots=blob-storage-quickstart-scratch) 를 참고하여 작성하였습니다.
{: .prompt-info}

## Spring Dependencies 추가

> SpringBoot v3.4.1, Java 17 을 기준으로 설정하였습니다.
{: .prompt-info}

다음과 같이 

1. spring-cloud-azure-dependencies : `v5.19.0`
2. spring-cloud-azure-starter-storage-blob : `v5.19.0`

의존성을 추가해주도록 하겠습니다.

```gradle
// https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-dependencies
implementation 'com.azure.spring:spring-cloud-azure-dependencies:5.19.0'

// https://mvnrepository.com/artifact/com.azure.spring/spring-cloud-azure-starter-storage-blob
implementation 'com.azure.spring:spring-cloud-azure-starter-storage-blob:5.19.0'
```

## 설정 파일

application.properties 또는 yml 파일의 구성을 다음과 같이 설정하여야 합니다. 설정을 제대로 하였다면, 따로 configuration 작성 없이도 바로 사용할 수 있습니다.

```yaml
spring:
  cloud:
    azure:
      storage:
        blob:
          account-name: {스토리지 계정의 이름}
          container-name: {컨테이너 이름}
          account-key: {스토리지 계정의 액세스 키}
```

참고로 스토리지 계정의 액세스 키는 포털에서 `스토리지 계정 > 보안 + 네트워킹 > 액세스 키` 경로에서 확인할 수 있습니다.

![img](/assets/img/2025-01-21-azure-blob-storage/3.png){: w="700" }
_스토리지 계정 액세스 키_

## 업로드

COASTER 서비스에서는 유저, 게시물, 채팅 각각의 이미지 파일을 디렉토리에 구분하여 저장하기 위해 DirName이라는 enum을 통하여 디렉토리 이름을 구분해주고 있습니다. 그리고 파일 이름은 중복을 피하기 위해 id으로 설정하였습니다.

특히 MultipartFile을 그대로 upload할 경우 content type 설정이 자동으로 이루어지지 않기 때문에, getUploadOptions() 메소드에서 파일의 타입에 맞게 헤더를 설정해주었습니다.

따라서 최종 코드는 다음과 같습니다.

```java
@Service
@RequiredArgsConstructor
public class BlobStorageService {
    private final BlobContainerClient blobContainerClient;

    @Value("${spring.cloud.azure.storage.blob.account-name}")
    private String account;

    @Value("${spring.cloud.azure.storage.blob.container-name}")
    private String container;

    public String upload(
            final MultipartFile file,
            final DirName imageDir,
            final Long id
    ) {
        String fileName = imageDir + "/" + id;

        BlobClient blobClient = blobContainerClient.getBlobClient(fileName);
        BlobParallelUploadOptions uploadOptions = getUploadOptions(file);
        blobClient.uploadWithResponse(uploadOptions, null, Context.NONE);

        // 클라이언트에게 url 형태로 반환하기 위함
        return String.format("https://%s.blob.core.windows.net/%s/%s/%s", account, container, imageDir, id);
    }

    private BlobParallelUploadOptions getUploadOptions(final MultipartFile file) {
        try {
            BlobHttpHeaders jsonHeaders = new BlobHttpHeaders()
                    .setContentType(file.getContentType());
            BinaryData data = BinaryData.fromStream(file.getInputStream(), file.getSize());
            return new BlobParallelUploadOptions(data)
                    .setRequestConditions(new BlobRequestConditions())
                    .setHeaders(jsonHeaders);
        } catch (IOException e) {
            throw new GeneralException(IO_EXCEPTION);
        }
    }
}
```

## 테스트

간단한 테스트를 위하여 다음과 같이 이미지 파일을 저장하고, url 값을 반환하는 api를 작성하였습니다.

```java
@RestController
@RequiredArgsConstructor
public class ImageController {
    private final BlobStorageService blobStorageService;

    @PostMapping("/api/image/test")
    public String imageUploadTest(
            @RequestParam("file") MultipartFile file
    ) {
        return blobStorageService.upload(file, DirName.USER, 1L);
    }
}
```

multipartFile 형태는 postman에서 body를 `form-data` 타입으로 설정한 뒤, 파일을 선택하여 실행해주면 됩니다.

![img](/assets/img/2025-01-21-azure-blob-storage/4.png){: w="600" }{: .shadow}
_postman 설정_

<br>

그리고 다시 azure portal에 돌아가서 컨테이너를 확인해보면? 다음과 같이 파일이 정상적으로 저장된 것을 확인할 수 있습니다.

![img](/assets/img/2025-01-21-azure-blob-storage/5.png){: w="600" }{: .shadow}
_사진 저장 확인 완료!_