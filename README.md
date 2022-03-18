# B2C-Custom-Policy

For Azure China, authentication endpoint need to updated as below

In the TrustFrameworkBase.xml, update the TechnicalProfile "login-NonInteractive":

```xml
<Item Key="ProviderName">https://sts.chinacloudapi.cn/</Item>
<Item Key="METADATA">https://login.chinacloudapi.cn/{tenant}/.well-known/openid-configuration</Item>
<Item Key="authorization_endpoint">https://login.chinacloudapi.cn/{tenant}/oauth2/token</Item>
```
