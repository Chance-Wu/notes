```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

  TextView responseText;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Button sendRequest = (Button) findViewById(R.id.send_request);
    responseText = (TextView) findViewById(R.id.response_text);
    sendRequest.setOnClickListener(this);
  }

  @Override
  public void onClick(View v) {
    if (v.getId() == R.id.send_request) {
      sendRequestWithHttpURLConnection();
    }
  }

  private void sendRequestWithHttpURLConnection() {
    //开启线程来发起网络请求
    new Thread(new Runnable() {
      @Override
      public void run() {
        HttpURLConnection connection = null;
        BufferedReader reader = null;
        try {
          //第一步获取HttpURLConnection的实例，一般只需new出一个URL对象，并传出目标的网络地址
          URL url = new URL("https://www.bai.com");

          //第二步调用openConnection()方法
          connection = (HttpURLConnection) url.openConnection();

          //第三步设置HTTP请求的方法，GET表示希望从服务器那里获取数据，POST表示希望提交数据给服务器
          connection.setRequestMethod("GET");

          //自由定制操作，可以设置连接超时、读取超时的毫秒数
          connection.setConnectTimeout(8000);
          connection.setReadTimeout(8000);

          //第四步调用getInputStream()方法获取服务器返回的数据流
          InputStream inputStream = connection.getInputStream();

          //第五步对获取到的输入流进行读取
          reader = new BufferedReader(new InputStreamReader(inputStream));
          StringBuilder response = new StringBuilder();
          String line;
          while ((line = reader.readLine()) != null) {
            response.append(line);
          }
          showResponse(response.toString());
        } catch (Exception e) {
          e.printStackTrace();
        } finally {
          if (reader != null) {
            try {
              reader.close();
            } catch (IOException e) {
              e.printStackTrace();
            }
          }
          if (connection != null) {
            //最后一步调用disconnect()方法关闭HTTP请求
            connection.disconnect();
          }
        }
      }
    }).start();
  }

  private void showResponse(final String response) {

    //调用runOnUiThread()方法将线程切换到主线程，然后在更新UI元素，因为不允许在子线程中更新UI操作
    runOnUiThread(new Runnable() {
      @Override
      public void run() {
        //在这里进行UI操作，将结果显示到界面上
        responseText.setText(response);
      }
    });
  }
}
```