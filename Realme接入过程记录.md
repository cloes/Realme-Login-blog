# Realme的Login接入过程记录

## 1.问题的提出

近期要使用Realme作为第三方接入，因为研究了一下Realme的请求过程（Request）和响应（Response），其中包含了一些加密和解密的方式。我们接下来分别从Request和Response两个方面进行讲述。

## 2.向Realme提交metadata

### 2.1根据模版修改sp-metadata.xml文件

我们在接入Realme之前，首先要向Realme平台提交我们的信息，其实就相当于把我们的应用注册到Realme。

我们可以从[https://developers.realme.govt.nz/assets/Uploads/Integration-Bundle-MTS-V2.1.zip](https://developers.realme.govt.nz/assets/Uploads/Integration-Bundle-MTS-V2.1.zip)下载相关的资源包。

我们以资源包中的MTSSP_PostBinding_Sample.xml作为模版进行修改，需要修改以下几个项目：

(1)**entityID**

	entityID: http://realme.test/mts2/sp

这个是我们接触Realme的唯一标识符，也就是ID。根据RealMe Messaging Specification Login V1.0.pdf第3.2.2 Element <Issuer>小节的说明：

<img src="pic1.png" alt="pic1.png" width="800px" />

entityID的值应该和AuthnRequest的Issuer的格式一致，Issuer格式要求请查询：[https://developers.realme.govt.nz/how-realme-works/realme-request-parameters/#login-service-authentication-request](https://developers.realme.govt.nz/how-realme-works/realme-request-parameters/#login-service-authentication-request)


(2)**Location**

	<md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://qacand.sflab.ondemand.com/saml2/SAMLAssertionConsumer?company=hmcscindiaec" index="0" isDefault="true" />

location属性就是IDP返回结果的时候，我们的服务器的接收地址，所以这个必须是公网可以访问的地址，不能使用内网的地址。
我们的配置如下：

	<md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://52.198.228.221/post.php" index="0" isDefault="true" />

(3)**Organization**

	<md:Organization>
		<md:OrganizationName xml:lang="en-us">
			Sample Service Provider
		</md:OrganizationName>
		<md:OrganizationDisplayName xml:lang="en-us">
			Sample Service Provider
		</md:OrganizationDisplayName>
		<md:OrganizationURL xml:lang="en-us">
			http://52.198.228.221/
		</md:OrganizationURL>
	</md:Organization>

这几个参数都是机构名称、机构显示名称、机构的URL这些参数，可以根据情况随便填。

(4)**ContactPerson**

	<md:ContactPerson contactType="technical">
		<md:Company>cloes</md:Company>
		<md:GivenName>cloes</md:GivenName>
		<md:SurName>375600284@qq.com</md:SurName>
	</md:ContactPerson>

这些是联系人信息，也可以根据情况随便填


一个完整的SP-metadata.xml例子如下：

	<md:EntityDescriptor ID="MTS SP" entityID="http://myrealme.test/mts2/sp" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:query="urn:oasis:names:tc:SAML:metadata:ext:query" xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" xmlns:xenc="http://www.w3.org/2001/04/xmlenc#">
		<md:SPSSODescriptor AuthnRequestsSigned="true" WantAssertionsSigned="true" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
			<md:KeyDescriptor use="signing">
				<ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
					<ds:X509Data>
						<ds:X509Certificate>
	MIIDgzCCAmugAwIBAgIEZ2ZIfzANBgkqhkiG9w0BAQsFADByMQswCQYDVQQGEwJO
	WjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5rbm93bjEQMA4GA1UEChMH
	VW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEbMBkGA1UEAxMSbXRzLmNsaWVudC5z
	aWduaW5nMB4XDTE2MDMyNDIxNTQzNFoXDTE5MDMyNDIxNTQzNFowcjELMAkGA1UE
	BhMCTloxEDAOBgNVBAgTB1Vua25vd24xEDAOBgNVBAcTB1Vua25vd24xEDAOBgNV
	BAoTB1Vua25vd24xEDAOBgNVBAsTB1Vua25vd24xGzAZBgNVBAMTEm10cy5jbGll
	bnQuc2lnbmluZzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANItT07w
	87HLup/Waw+Ek7S6RlJEu9xn1UXVfL84dkLufy21Cklnyzxzd2op9FbUq68jalIa
	9hlguI9B7QqLplIcNOfhTWT+CFz9j0LL3PyOpl+/oDKJdwUPwtzz312fqfWz/vqZ
	Rz9cx0kmJlHUdz97WeD0Iem3px/4lM6nZtJWXF+0olZO5PxNnG/X8W9Y4qPT00FY
	AyHh0s2PUjbOuB9LAf4oWFxzNiluT5vifXLo0AR2u0m46dxxDbplX+G17/Ozs5Pv
	Ew/rBeUsLlgS862KClir16iw43TY6cTuw1fHUmqXuywrFwuKZUi5FX5yahGjKVxP
	oI/xg8QYDJASA5UCAwEAAaMhMB8wHQYDVR0OBBYEFFjGNNt5tDt6KqJcmo/HAv7F
	E2aeMA0GCSqGSIb3DQEBCwUAA4IBAQB1TvX7QbikPh/J2auRT+MWrScwDh9KQ/7c
	VysDKTqGFvQWm+t38hFwjy8DG6/GWzMmlvPzs+cpxxrU4wYINIki3q2XRMU5pNDS
	EWTqvcz+Of4WCNf0Kl6vaiQMo3AGhz36VemJeatiKTMnId9b//QKwG5qx9SGDKmY
	QOpm6Rtvai8oMhIX7Zgdv1pyVOIFvFbj8gC1U5krrwufSjTIZD8Vi0KCVYXLMXad
	BQj0IlNkxjIYgqj0oNhwemLs2vcXm/emYVjIekJVVgaY7E+pUTOkGQqJlLWP5p/G
	4QhF24m0cEw6CaJVbXOuxQPo7u27TCM8FM1SBZEaFdOtVVB8rxO5
						</ds:X509Certificate>				
					</ds:X509Data>
				</ds:KeyInfo>
			</md:KeyDescriptor>
			<md:KeyDescriptor use="encryption">
				<ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
					<ds:X509Data>
						<ds:X509Certificate>
	MIIDgzCCAmugAwIBAgIEZ2ZIfzANBgkqhkiG9w0BAQsFADByMQswCQYDVQQGEwJO
	WjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5rbm93bjEQMA4GA1UEChMH
	VW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEbMBkGA1UEAxMSbXRzLmNsaWVudC5z
	aWduaW5nMB4XDTE2MDMyNDIxNTQzNFoXDTE5MDMyNDIxNTQzNFowcjELMAkGA1UE
	BhMCTloxEDAOBgNVBAgTB1Vua25vd24xEDAOBgNVBAcTB1Vua25vd24xEDAOBgNV
	BAoTB1Vua25vd24xEDAOBgNVBAsTB1Vua25vd24xGzAZBgNVBAMTEm10cy5jbGll
	bnQuc2lnbmluZzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANItT07w
	87HLup/Waw+Ek7S6RlJEu9xn1UXVfL84dkLufy21Cklnyzxzd2op9FbUq68jalIa
	9hlguI9B7QqLplIcNOfhTWT+CFz9j0LL3PyOpl+/oDKJdwUPwtzz312fqfWz/vqZ
	Rz9cx0kmJlHUdz97WeD0Iem3px/4lM6nZtJWXF+0olZO5PxNnG/X8W9Y4qPT00FY
	AyHh0s2PUjbOuB9LAf4oWFxzNiluT5vifXLo0AR2u0m46dxxDbplX+G17/Ozs5Pv
	Ew/rBeUsLlgS862KClir16iw43TY6cTuw1fHUmqXuywrFwuKZUi5FX5yahGjKVxP
	oI/xg8QYDJASA5UCAwEAAaMhMB8wHQYDVR0OBBYEFFjGNNt5tDt6KqJcmo/HAv7F
	E2aeMA0GCSqGSIb3DQEBCwUAA4IBAQB1TvX7QbikPh/J2auRT+MWrScwDh9KQ/7c
	VysDKTqGFvQWm+t38hFwjy8DG6/GWzMmlvPzs+cpxxrU4wYINIki3q2XRMU5pNDS
	EWTqvcz+Of4WCNf0Kl6vaiQMo3AGhz36VemJeatiKTMnId9b//QKwG5qx9SGDKmY
	QOpm6Rtvai8oMhIX7Zgdv1pyVOIFvFbj8gC1U5krrwufSjTIZD8Vi0KCVYXLMXad
	BQj0IlNkxjIYgqj0oNhwemLs2vcXm/emYVjIekJVVgaY7E+pUTOkGQqJlLWP5p/G
	4QhF24m0cEw6CaJVbXOuxQPo7u27TCM8FM1SBZEaFdOtVVB8rxO5
						</ds:X509Certificate>				
					</ds:X509Data>
				</ds:KeyInfo>
			</md:KeyDescriptor>
			<md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="http://52.198.228.221/post.php" index="0" isDefault="true" />
		</md:SPSSODescriptor>
		<md:Organization>
			<md:OrganizationName xml:lang="en-us">
				Sample Service Provider
			</md:OrganizationName>
			<md:OrganizationDisplayName xml:lang="en-us">
				Sample Service Provider
			</md:OrganizationDisplayName>
			<md:OrganizationURL xml:lang="en-us">
				http://52.198.228.221/
			</md:OrganizationURL>
		</md:Organization>
		<md:ContactPerson contactType="technical">
			<md:Company>cloes</md:Company>
			<md:GivenName>cloes</md:GivenName>
			<md:SurName>375600284@qq.com</md:SurName>
		</md:ContactPerson>
	</md:EntityDescriptor>


### 2.2上传sp的metadata文件

sp的metadata文件上传地址：[https://mts.realme.govt.nz/logon-mts/metadataupdate](https://mts.realme.govt.nz/logon-mts/metadataupdate)

然后如果通过格式校验后，就会显示如下的页面：

<img src="pic2.png" alt="pic2.png" width="800px" />

点击“import”后，就会显示如下页面：

<img src="pic3.png" alt="pic3.png" width="800px" />


## 3.创建AuthnRequest请求

### 3.1 AuthnRequest的例子
我们可以在RealMe Messaging Specification Login V1.0.pdf中的3.6 Sample Request中找到AuthnRequest的例子：
	
	<samlp:AuthnRequest
	    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
	    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
		AssertionConsumerServiceIndex="0"
		Destination="https://realme.govt.nz/sso/SSORedirect/metaAlias/logon-idp"
		ID="a958a20e059c26d1cfb73163b1a6c4f9"
		IssueInstant="2012-05-21T00:39:32Z"
		ProviderName="Sample Service Provider"
		Version="2.0">
		    <saml:Issuer>https://www.sample-client.co.nz/onlineservices/service1</saml:Issuer>
		    <samlp:NameIDPolicy AllowCreate="true" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"></samlp:NameIDPolicy>
		    <samlp:RequestedAuthnContext>
		        <saml:AuthnContextClassRef>
				urn:nzl:govt:ict:stds:authn:deployment:GLS:SAML:2.0:ac:classes:ModStrength
				</saml:AuthnContextClassRef>
		    </samlp:RequestedAuthnContext>
	</samlp:AuthnRequest>


### 3.2 对AuthnRequest的xml数据进行处理

根据saml-bindings-2.0-os.pdf中3.4.4 Message Encoding的描述，我们需要对AuthnRequest按照以下顺序进行操作:

- DEFLATE compression
- base64-encoded
- URL-encoded

具体可以使用Realme-Go-SDK([https://github.com/cloes/Realme-GO-SDK](https://github.com/cloes/Realme-GO-SDK))

**注意：**


- 在这里ID的规则必须以字母"a"开头，例如上面例子中的ID="a958a20e059c26d1cfb73163b1a6c4f9"

- issueInstant为AuthnRequest的生成时间，有效时间只有60s，如果超时的话，就需要重新生成issueInstant时间

- issueInstant使用的是UTC时间，格式是2012-05-21T00:39:32Z

### 3.3 向Realme的测试平台提交AuthnRequest

#### 3.3.1 向MTS平台提交测试请求

我们可以向Realme的测试地址提交POST请求，对我们的AuthnRequest进行测试。

测试地址：https://mts.realme.govt.nz/logon-mts/mtsEntryPoint

例如，我们经过上述的数据处理后，我们得到的url地址是：

	https://mts.realme.govt.nz/logon-mts/mtsEntryPoint?SAMLRequest=fZJbbxoxEIWfm19h%2BR32QkLCiF0JQVshJRUKVR7y5ngHWMlrbz2zFPLra3ORNlLFq%2BccnW%2FGZ0qqMS3MOt7ZV%2FzTIbE4NMYSxEEhO2%2FBKaoJrGqQgDWsZy%2FPkA9TUETouXZW9iztbU%2FrHTvtjBSzq3vuLHUN%2BjX6fa1xaSs8FDKVYhFgaquippA75pYgSRqmoUdlGhxu3Z6H9jMxbuvsIAzi8Ltlf1y52rIUy0Uh1eThSeUppg8TnY%2BrTG8%2BHkfZePSRqbG%2B30yCiqgLqcTKciHzNHsapPkgG%2F9OR5A%2Fwn32LsXKu31dof8VFjqzRJTjBYQDaMzOE2qleENPJ%2BSwryzvhJjGu8Apxpe3vNOkr7w6W4ipy8XKmVofxcwY93ce3BxI2HcoxQ%2FnG8W3Dx9f6mqwOUmhjYzEGK5UnlO%2FpvTCL63A6tSR8FmMBy7vvp236j%2FOTWjEK27KyGE%2FDcQPglozEFcEKkqhwta4YxOS4efzulcmDTr6A%2FaLq9bs0W55d7nIf1MiYnKT8Trut7v8Bw%3D%3D&SigAlg=http%3A%2F%2Fwww.w3.org%2F2000%2F09%2Fxmldsig%23rsa-sha1&Signature=0U0j1l0a5rwCMWxN8fcU%2BJsLjAJLiTTZgBVWRdz3GkJ%2BojkeGWFVcyGgUepRWYyC%2FlemTluAKpHOAXr1INg2luskaZ%2BWSpQRKUC6t68zEzuZSkhkxnp1YiOWMNlqMCfAT0OdxlADqvwGd2qzCeBN5FxJ0cdqpNdy7g%2BP%2B6PukB8wOTGWqp0AfAaAK506ZDiuvZofg8rotIxYYmuloA44mOiQw5D5aV04FkThZSPBsZ9XvEXeB4aKHFpaTRsguxxsUNc8tttuW5aUJsNwunU%2Fk2ybxYYiA80CSqZR5qTFc7DP%2Fz90DtvzADXoyrMTpKGJp6pWWobNRly9g17IJxtqoA%3D%3D

在有效时间内，访问上述地址，测试系统会显示如下页面：

<img src="pic5.png" alt="pic5.png" width="800px" />

#### 3.3.2 向MTS的在线校验平台验证AuthnRequest的格式正确性（可选步骤）

我们可以先在线验证我们的AuthnRequest请求格式，确定了格式的正确性后，我们再向正式的MTS平台发起请求。

在线校验地址：https://mts.realme.govt.nz/logon-mts/authnrequestsubmit

如果验证不通过的话，它会显示相关的错误信息，如图：

<img src="pic4.png" alt="pic4.png" width="800px" />

如果验证通过，我们会得到一下的页面：

<img src="pic5.png" alt="pic5.png" width="800px" />


# 4.对Response响应进行解密








