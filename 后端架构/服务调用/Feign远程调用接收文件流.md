Feign客户端接口方法接收Response

```java
@FeignClient(value = "service-file")
public interface FileDataService {
  /**
   * 根据key获取文件输入流
   * @param fileUuid
   * @return
   * 这里Response 依赖feign的 import feign.Response;
   */
  @RequestMapping(value = "/innerApi/getFileInputStream",method = RequestMethod.GET)
  Response getFileInputStream(@RequestParam String fileUuid);
}
```



服务提供方：

```java
@ApiOperation(value = "获取文件输入流/innerApi:getFileInputStream")
@GetMapping(value = "/getFileInputStream")
@ApiOperationSupport(order = 1)
public void  getFileInputStream(@RequestParam String fileUuid, HttpServletResponse response){
  try {
    Map<String,Object> map = new HashMap<>();
    map.put("fileUuid",fileUuid);
    List<FileStudMid> list  = fileStudMidService.selectByMap(map);
    if(null!=list && list.size()==1){
      FileStudMid fileStudMid = list.get(0);
      String filename = fileStudMid.getFileName();
      String objectKey = fileStudMid.getFilePath();
      response.setContentType(FileContentTypeEnum.getMimeType(fileStudMid.getFileType()));
      response.addHeader("Content-Disposition", "attachment;filename=" + new String(filename.getBytes(), "ISO-8859-1"));
      OutputStream out = response.getOutputStream();
      InputStream inputStream = OssClientUtil.downloadObsClientFile(OssClientUtil.OSS_BUCKET_NAME, objectKey);
      byte[] buffer = new byte[2048];
      int i = -1;
      while ((i = inputStream.read(buffer)) != -1) {
        out.write(buffer, 0, i);
      }
      out.flush();
      out.close();
      inputStream.close();
    }
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



最后调用服务时获取的response转成inputStream：

```java
Response fileInputStream = fileDataService.getFileInputStream(fileUUid);
InputStream inputStream = fileInputStream.body().asInputStream();
byte[] buffer = new byte[2048];
......
```