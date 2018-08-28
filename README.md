# APNS in PHP

``` php
<?php

/**
 * 消息推送类
 */

class SureApns
{
	// curl对象
	private $m_curl;
	
	function __construct ($iApnsType, $iApnsEnvType) {
	    $this->m_curl = curl_init();
	    curl_setopt($this->m_curl, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_2_0);
	    curl_setopt($this->m_curl, CURLOPT_RETURNTRANSFER, 1);
	    curl_setopt($this->m_curl, CURLOPT_TIMEOUT, 5);
	    $this->m_sAppId = "apns-topic:com.test.sharebook";
	    curl_setopt($this->m_curl, CURLOPT_SSLCERT, "/data/cfg/push/apns_sharebook_sandbox.pem");
	    curl_setopt($this->m_curl, CURLOPT_SSLCERTPASSWD, "111111");
	}

	/**
	 * apns消息推送，针对设备号
	 * @param  string  $sDevice     设备号
	 * @param  string  $sMsg        推送提醒
	 * @param  array   $arrContent  推送内容数组
	 * @param  integer $iExpireTime 过期时间，默认一天
	 * @return boolean              是否成功
	 */
    function pushMsg ($sDevice, $sMsg, $arrContent = array(), $iExpireTime = 86400) {
    
        $iBadge = 1;

        $arrPayload["aps"] = array(
            'alert'    => $sMsg,
	        'content'  => $arrContent,
	        'badge'    => $iBadge,
	        'sound'    => 'default',
	        'content-available' => 1
        );

		// 开发环境
		$sHttp = "https://api.development.push.apple.com/3/device/".$sDevice;

		// 生产环境
		// $sHttp = "https://api.push.apple.com/3/device/".$sDevice;

		curl_setopt($this->m_curl, CURLOPT_URL, $sHttp);
		curl_setopt($this->m_curl, CURLOPT_POSTFIELDS, json_encode($arrPayload));
		curl_setopt($this->m_curl, CURLOPT_HTTPHEADER, array($this->m_sAppId, "apns-expiration:".$iExpireTime));

		$sRet = curl_exec($this->m_curl);
		$iCode = curl_getinfo($this->m_curl, CURLINFO_HTTP_CODE);

		if ($iCode == 410) {
			// 说明设备存在问题
			SURE_LOG(__FILE__, __LINE__, LP_ERROR, "{$sDevice} 存在问题 -- ".$iCode);
			return false;
		}

		if ($iCode != 200) {
			SURE_LOG(__FILE__, __LINE__, LP_ERROR, "{$sHttp} --> {$iCode} --> {$sRet}");
			return false;
		}

		return true;

	}

	/**
	 * 析构函数
	 */
	function __destruct () {
		if (isset($this->m_curl)) {
			curl_close($this->m_curl);
			unset($this->m_curl);
		}
	}

}

```