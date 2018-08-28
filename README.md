# APNS in PHP

``` php
<?php

$curl = curl_init();
curl_setopt($curl, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_2_0);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($curl, CURLOPT_TIMEOUT, 5);
$sAppId = "apns-topic:com.test.sharebook";
curl_setopt($curl, CURLOPT_SSLCERT, "/data/cfg/push/apns_sharebook_sandbox.pem");
curl_setopt($curl, CURLOPT_SSLCERTPASSWD, "111111");

$arrPayload["aps"] = array(
    'alert'    => $sMsg,
    'content'  => $arrContent,
    'badge'    => 1,
    'sound'    => 'default',
    'content-available' => 1
);

// 开发环境
$sHttp = "https://api.development.push.apple.com/3/device/".$sDevice;

// 生产环境
// $sHttp = "https://api.push.apple.com/3/device/".$sDevice;

curl_setopt($curl, CURLOPT_URL, $sHttp);
curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($arrPayload));
curl_setopt($curl, CURLOPT_HTTPHEADER, array($sAppId, "apns-expiration:".$iExpireTime));

$sRet = curl_exec($curl);
$iCode = curl_getinfo($curl, CURLINFO_HTTP_CODE);

if ($iCode == 410) {
// 说明设备存在问题
return;
}

if ($iCode != 200) {
return;
}

curl_close($curl);
unset($curl);

```