---
layout: post
title:  "PHP에서 cURL을 비동기로 사용하기"
date:   2016-10-18 00:00:00
categories: php
tags: curl
comments: true
---

일반적으로 cURL을 이용하여 HTTP 프로토콜 데이터를 POST 방식으로 보내는 PHP 함수는 다음과 같습니다.

``` php

function https_post($uri, $postdata)  
{  
    $ch = curl_init($uri);  
    curl_setopt($ch, CURLOPT_POST, true);  
    curl_setopt($ch, CURLOPT_POSTFIELDS, $postdata);  
    $result = curl_exec($ch);  
    curl_close($ch);  
    return $result;  
} 

```

하지만, 이 방식은 결과값이 전달 될 때 까지 블럭(block) 된다는 것이 문제입니다.   
따라서 결과값이 불필요한 경우에는 요청을 보내고 잊어버리면 그만입니다.   
아쉽게도 PHP의 cURL 라이브러리는 비동기 처리를 지원하지 않습니다. 따라서 직접 소켓을 오픈하여 요청을 보내야 합니다.

``` php
function curl_request_async($url, $params, $type='POST')  
{  
    foreach ($params as $key => &$val)  
    {  
        if (is_array($val))  
            $val = implode(',', $val);  
        $post_params[] = $key.'='.urlencode($val);  
    }  
    $post_string = implode('&', $post_params);  
  
    $parts=parse_url($url);  
  
    if ($parts['scheme'] == 'http')  
    {  
        $fp = fsockopen($parts['host'], isset($parts['port'])?$parts['port']:80, $errno, $errstr, 30);  
    }  
    else if ($parts['scheme'] == 'https')  
    {  
        $fp = fsockopen("ssl://" . $parts['host'], isset($parts['port'])?$parts['port']:443, $errno, $errstr, 30);  
    }  
  
    // Data goes in the path for a GET request  
    if('GET' == $type)  
        $parts['path'] .= '?'.$post_string;  
  
    $out = "$type ".$parts['path']." HTTP/1.1\r\n";  
    $out.= "Host: ".$parts['host']."\r\n";  
    $out.= "Content-Type: application/x-www-form-urlencoded\r\n";  
    $out.= "Content-Length: ".strlen($post_string)."\r\n";  
    $out.= "Connection: Close\r\n\r\n";  
    // Data goes in the request body for a POST request  
    if ('POST' == $type && isset($post_string))  
        $out.= $post_string;  
  
    fwrite($fp, $out);  
    fclose($fp);  
}  
```

위 코드는 인터넷에 부유 중인 것을 HTTPS (HTTP Secure) 프로토콜도 지원하도록 수정 한 것입니다. 이 방법을 이용하면 전송 할 데이터를 소켓으로 방출 한 후에 곧바로 함수가 리턴됩니다. 

즉, 데이터를 받기 위한 추가적인 처리를 프로세스가 감당 할 필요가 없다는 것을 의미합니다.

만약 빠른 응답 속도를 보장 받는 것이 최우선 과제라면 다음과 같이 프로세스를 백그라운드로 실행 하게 만들 수도 있겠습니다. 

단, 아래 코드는 POSIX 계열에서만 사용 가능합니다.

``` php
function curl_post_async($uri, $params)  
{  
        $command = "curl ";  
        foreach ($params as $key => &$val)  
                $command .= "-F '$key=$val' ";  
        $command .= "$uri -s > /dev/null 2>&1 &";  
        passthru($command);  
}  
```
이 방식은 지체 없이 curl 프로세스를 백그라운드로 구동하기 때문에 소요 시간이 거의 없다고 볼 수 있습니다.