---
layout: post
title:  "스프링 파일 업로드"
date:   2021-05-26 23:10
categories: spring, fileUpload
tags: [java, fileUpload]
---

### 스프링 파일 업로드하기

### build.gradle 추가 

```
    compile group: 'commons-io', name: 'commons-io', version: '2.6' /* Apache Commons IO */
    compile group: 'commons-fileupload', name: 'commons-fileupload', version: '1.3.3' /* Apache Commons File Upload */

```

### JS to Server

```java
$("input[type='file']").change(function (e) {
        let formData = new FormData();
        let inputFile = $("input[name='uploadFile']");
        let files = inputFile[0].files;

        for (let i = 0; i < files.length; i++) {
            if (!checkExtension(files[i].name, files[i].size)) {
                return false;
            }

            formData.append("uploadFile", files[i]);
        }

        $.ajax({
            url: '/uploadAjaxAction',
            processData: false,
            contentType: false,
            data: formData,
            type: 'POST',
            // dataType: 'json',
            success: function (result) {
                console.log("# result");
                console.log(result);
            }
        });

    });
```

### 컨트롤러

```java

    @PostMapping(value = "/uploadAjaxAction")
    public void uploadAjaxPost(MultipartFile[] uploadFile){
        for (MultipartFile multipartFile : uploadFile) {
            log.info("================");
            log.info("uploadfileName " + multipartFile.getOriginalFilename());
            log.info("uploadfilesize " + multipartFile.getSize());

            String uploadFileName = multipartFile.getOriginalFilename();
            File saveFile = new File(uploadFolder, uploadFileName);
            try {
                multipartFile.transferTo(saveFile);
            } catch (Exception e) {
                log.error(e.getMessage());
            }
        }
   }
```

### 이미지 업로드 시 고려사항
- 동일한 이름으로 업로드 시 기존 파일이 삭제되는 경우
- 이미지 파일의 경우 원본 파일의 용량이 큰 경우 섬네일 이미지를 생성해야 하는 문제
- 첨부파일 공격에 대한 업로드 파일의 확장자 제한

> 중복된 이름의 첨부파일 처리
    - UUID를 활용
```java
    @PostMapping(value = "/uploadAjaxAction")
    public void uploadAjaxPost(MultipartFile[] uploadFile){
        for (MultipartFile multipartFile : uploadFile) {
            log.info("================");
            log.info("uploadfileName " + multipartFile.getOriginalFilename());

            File uploadPath = new File(uploadFolder,getFolder());
            log.info("# getFolder() " +getFolder());

            if(!uploadPath.exists()) {
                uploadPath.mkdirs();
            }


            String uploadFileName = multipartFile.getOriginalFilename();

            UUID uuid = UUID.randomUUID();
            uploadFileName = uuid + "_" + uploadFileName;
            File saveFile = new File(uploadPath, uploadFileName);
            try {
                multipartFile.transferTo(saveFile);
            } catch (Exception e) {
                log.error(e.getMessage());
            }
        }
   }
```

> 한 폴더 내에 많은 파일 생성
    - 년/월/일 폴더 생성처리
```java
   private String getFolder() {
       SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
       final Instant instant = LocalDate.now().atStartOfDay(ZoneId.systemDefault()).toInstant();
       Date date = Date.from(instant);
       String str = sdf.format(date);
       log.info("===getFolder");
       log.info(str);
       return str.replace("-", File.separator);
   }
```

### 서버에서 브라우저로 정보 설계
    - 업로드 된 파일의 이름과 원본 파일의 이름
    - 파일이 저장된 경로
> 객체 생성해서 처리 DTO

```java
 @PostMapping(value = "/uploadAjaxAction")
    @ResponseBody
    public ResponseEntity<List<AttachFileDto>> uploadAjaxPost(MultipartFile[] uploadFile){
        List<AttachFileDto> list = new ArrayList<>();
        String uploadFolderPath = getFolder();
        File uploadPath = new File(uploadFolder, uploadFolderPath);

        if(!uploadPath.exists()) {
            uploadPath.mkdirs();
        }

        for (MultipartFile multipartFile : uploadFile) {
            AttachFileDto attachFileDto = new AttachFileDto();
            String uploadFileName = multipartFile.getOriginalFilename();
            UUID uuid = UUID.randomUUID();

            uploadFileName = uuid + "_" + uploadFileName;

            final AttachFileDto bindingDto = attachFileDto.makeBinding(uploadFileName, uploadFolderPath, uuid.toString());

            try {
                File saveFile = new File(uploadPath, uploadFileName);
                multipartFile.transferTo(saveFile);
                FileOutputStream thumbnail = new FileOutputStream(new File(uploadPath, "s_" + uploadFileName));
                Thumbnailator.createThumbnail(multipartFile.getInputStream(), thumbnail, 100, 100);

                thumbnail.close();
            } catch (Exception e) {
                log.error(e.getMessage());
            }
            list.add(bindingDto);
        }

        return new ResponseEntity<>(list, HttpStatus.OK);
   }
```

> 특정한 파일 이름을 받아 이미지 데이터를 전송하는 메서드
```java
    @GetMapping("/display")
    public ResponseEntity<byte[]> getFile(String fileName) throws IOException {
        log.info("fileName {}", fileName);
        File file = new File(uploadFolder + fileName);
        log.info("file {}", file);

        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Type", Files.probeContentType(file.toPath()));
        return new ResponseEntity<>(FileCopyUtils.copyToByteArray(file), headers, HttpStatus.OK);

    }

```

### 이미지 삭제
- 파일 삭제 후 브라우저에서도 삭제
```java
    @PostMapping("/deleteFile")
    public ResponseEntity<String> deleteFile(String fileName) throws UnsupportedEncodingException {
        log.info("fileName {}", fileName);

        File file;
        file = new File(uploadFolder + URLDecoder.decode(fileName, "UTF-8"));
        file.delete();

        return new ResponseEntity<>("deleted", HttpStatus.OK);
    }
```
- 비정상적으로 브라우저 종료 시 업로드 된 파일의 처리 
    - quartz 라이브러리 활용
