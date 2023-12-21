---
layout: post
title: "(Wechall) MySQL I"
date: 2021-01-13 05:24:00 -0400
categories: Security CTF
---

# (Wechall) MySQL 1 write up

아래 주어진 코드를 통해 쿼리문을 살펴보면 쿼리문에 대해 별다른 처리가 되지 않은 것을 알 수 있다. (prepared statement x)
따라서 사용자 입력 값에 따라 DB를 조회하는 SQL injection 취약점이 존재한다.

![image](https://user-images.githubusercontent.com/24788751/104435989-66baaa80-55d0-11eb-9efb-4180f83373eb.png)

위 값을 입력하여 admin 계정으로 시스템에 로그인을 시도했다.
admin은 관리자 계정, '는 username 값이 끝났음을 명시한다. 주석처리#을 통해서 이후 오는 조건절을 무시하도록 한다.

```java
/**
 * Exploit this!
 * @param WC_Challenge $chall
 * @param unknown_type $username
 * @param unknown_type $password
 * @return boolean
 */
function auth1_onLogin(WC_Challenge $chall, $username, $password)
{
	$db = auth1_db();
	
	$password = md5($password);
	
	$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
	
	if (false === ($result = $db->queryFirst($query))) {
		echo GWF_HTML::error('Auth1', $chall->lang('err_unknown'), false); # Unknown user
		return false;
	}

	# Welcome back!
	echo GWF_HTML::message('Auth1', $chall->lang('msg_welcome_back', htmlspecialchars($result['username'])), false);
	
	# Challenge solved?
	if (strtolower($result['username']) === 'admin') {
		$chall->onChallengeSolved(GWF_Session::getUserID());
	}
	
	return true;
}
```