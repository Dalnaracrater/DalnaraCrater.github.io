---
layout: post
title: (Wechall) MySQL II
date: 2021-01-13 05:24:00
tags: Security CTF
---

# (Wechall) MySQL 2 write up
아래 주어진 코드를 살펴보면 MySQL 1과 달리 username에 대해 먼저 데이터를 조회한 뒤, 결과 값에 password 입력을 조회하여 사용자를 인증한다. 이때 password는 md5 암호화 해시 알고리즘에 의해 암호화된다.

UNION 키워드를 이용하면 두 개의 쿼리문을 하나로 합칠 수 있다. 다시 말해 테이블을 사용자 마음대로 재구성할 수 있다.

여러 시도 끝에 아래와 같이 입력 값을 구성하면 사용자 인증에 성공할 수 있다.

Username: ' UNION SELECT 1, 'admin', md5(1234)#

Password: 1234

Username에 입력된 값을 통해 기존 쿼리문의 데이터는 아무 것도 없고, 이후의 서브 쿼리를 통해 1, 'admin', md5(1234)를 레코드로 갖는 테이블이 생성된다.

```java
/**
 * Exploit this! It is the same as MySQL-I, but with an additional check, marked with ###
 * @param WC_Challenge $chall
 * @param unknown_type $username
 * @param unknown_type $password
 * @return boolean
 */
function auth2_onLogin(WC_Challenge $chall, $username, $password)
{
	$db = auth2_db();
	
	$password = md5($password);
	
	$query = "SELECT * FROM users WHERE username='$username'";
	
	if (false === ($result = $db->queryFirst($query))) {
		echo GWF_HTML::error('Auth2', $chall->lang('err_unknown'), false);
		return false;
	}
	
	
	#############################
	### This is the new check ###
	if ($result['password'] !== $password) {
		echo GWF_HTML::error('Auth2', $chall->lang('err_password'), false);
		return false;
	} #  End of the new code  ###
	#############################
	
	
	echo GWF_HTML::message('Auth2', $chall->lang('msg_welcome_back', array(htmlspecialchars($result['username']))), false);
	
	if (strtolower($result['username']) === 'admin') {
		$chall->onChallengeSolved(GWF_Session::getUserID());
	}
	
	return true;
}
```