# xcx_qrcode
TP5生成小程序二维码

小程序wxml

```html
<image src='{{toimg}}' class="erweima {{showewm?'':'hidden'}}" bindtap='clerkJurisdiction1' data-of='{{off}}'></image>
```

小程序js

```js
  //二维码
  clerkJurisdiction1: function (e) {
    var that = this;
    if (e.currentTarget.dataset.of == 1) {
      wx.previewImage({
        current: that.data.toimg, // 当前显示图片的http链接
        urls: [that.data.toimg] // 需要预览的图片http链接列表
      })
    } else {
      //生成分享二维码
      wx.request({
        url: CONFIG.API_URL.qrcode,
        data: {
          openId: app.globalData.userInfo.openid,
        },
        method: 'POST',
        header: {
          'content-type': 'application/x-www-form-urlencoded'
        },
        success: function (res) {
          var toimg = that.data.URL + res.data.qrcodeurl;
          that.setData({
            toimg: toimg,
            off: 1,
          })
        }
      });
    }
  },
```


后台控制器php代码

```php
    /*********************zly */
    /*
    2017年9月29日15:30:38
    生成带参数的二维码
    */
    public function qrcode()
    {
        $openid = input('param.openId');
        //1.获取token
        //从后台获取x_appid
        //获取x_appsecret
        $config = Db::name('wx_config')->where('id', 'eq', '1')->find();
        $appid = $config['x_appid'];
        $appsecret = $config['x_appsecret'];

        //获取token
        $tokenUrl="https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=$appid&secret=$appsecret";
        $getArr=array();
        $tokenArr=json_decode($this->send_post($tokenUrl, $getArr, "GET"));
        $access_token=$tokenArr->access_token;

        //通过token 获取小程序二维码
        $path="pages/index/index?openid=".$openid;
        $width=100;//二维码大小
        $post_data='{"path":"'.$path.'","width":'.$width.',"scene":"'.$openid.'"}';
        //$url="https://api.weixin.qq.com/cgi-bin/wxaapp/createwxaqrcode?access_token=".$access_token; //二维码2
            $url = "https://api.weixin.qq.com/wxa/getwxacode?access_token=".$access_token; //二维码1
            $result=$this->api_notice_increment($url, $post_data);

        $filename = $openid.'.jpg'; //图片 名称
            
        //二进制流转换成图片
            $jpg = $result;//得到post过来的二进制原始数据
            $file = fopen(ROOT_PATH . 'public' . DS . 'uploads/qrcode/'.$filename, "w");//打开文件准备写入
            fwrite($file, $jpg);//写入
            fclose($file);//关闭

            //返回二维码路径
            $qrcodeurl = DS . 'uploads/qrcode/'.$filename;
            Db::name('wx_user')->where('openid','eq',$openid)->update(['qrcodeurl'=>$qrcodeurl]);
        
        return json_encode(array('qrcodeurl'=>$qrcodeurl,'status'=>1, 'message'=>'生成成功'));
    }

    public function api_notice_increment($url, $data){
        $ch = curl_init();
        //$header = "Accept-Charset: utf-8";
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);
        //curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
        curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/5.0 (compatible; MSIE 5.01; Windows NT 5.0)');
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
        curl_setopt($ch, CURLOPT_AUTOREFERER, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        $tmpInfo = curl_exec($ch);
        if (curl_errno($ch)) {
            return false;
        }else{
            return $tmpInfo;
        }
    }

    public function send_post($url, $post_data,$method='POST') {
        $postdata = http_build_query($post_data);
        $options = array(
            'http' => array(
            'method' => $method, //or GET
            'header' => 'Content-type:application/x-www-form-urlencoded',
            'content' => $postdata,
            'timeout' => 15 * 60 // 超时时间（单位:s）
        )
        );
        $context = stream_context_create($options);
        $result = file_get_contents($url, false, $context);
        return $result;
    }
```
