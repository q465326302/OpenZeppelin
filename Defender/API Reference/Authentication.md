# API Authentication
Defender APIs使用短暂的JWT代币进行身份验证，可以通过[SRP协议](https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol)进行协商。我们建议使用[Amazon Cognito User Pool SDK](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-integrate-apps.html)来协商代币。

JWT代币将在60分钟后过期。如果您的代码需要长于60分钟的会话，请考虑重新创建JWT代币或使用[刷新代币](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)。

> NOTE
如果您使用defender-client npm包，所有身份验证和续订都会自动处理，只需提供API密钥和密钥即可。
## 进行身份验证的请求

一旦获得JWT代币，即可向Defender API发出请求。请求需要API密钥，JWT代币（可选）有效载荷和API URL。将$KEY，$TOKEN设置为之前获取的API密钥和JWT代币的值。$END_POINT可以是txs或sign。
```
API_URL='http://api.defender.openzeppelin.com/'

curl \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "$API_URL/$END_POINT"
```

## 使用Python
官方的[AWS SDK](https://aws.amazon.com/sdk-for-python/) 的 python不支持SRP身份验证，但是可以使用warrant库检索JWT[代币](https://github.com/capless/warrant#cognito-srp-utility)认证。
```
import boto3
from warrant.aws_srp import AWSSRP

client = boto3.client('cognito-idp', region_name='us-west-2')
aws = AWSSRP(username='API_KEY', password='API_SECRET', pool_id='POOL_ID', client_id='CLIENT_ID', client=client)
tokens = aws.authenticate_user()
print('Access Token', tokens['AuthenticationResult']['AccessToken'])
```

请将API_KEY和API_SECRET替换为您的API密钥和密钥，将POOL_ID和CLIENT_ID替换为您正在访问的API的用户池ID。

## API设置
这些设置由defender-client包自动管理，但如果您手动处理身份验证，则需要它们来配置SRP协议。

* 管理员API位于defender-api.openzeppelin.com主机上，身份验证由用户池us-west-2_94f3puJWv和客户端40e58hbc7pktmnp9i26hh5nsav提供。您将需要团队API密钥进行身份验证，可以在Defender的右上角菜单中找到。

*  Relayer API位于api.defender.openzeppelin.com主机上，身份验证由用户池us-west-2_iLmIggsiy和客户端1bpd19lcr33qvg5cr3oi79rdap提供。您将需要生成 Relayer API密钥进行身份验证，这些密钥在每个 Relayer 的页面中创建。