# B2C-Custom-Policy

IdentityExperienceFramework
https://xuzhoub2c.b2clogin.cn/xuzhoub2c.partner.onmschina.cn


Update "TenantId" and "PublicPolicyUri" in the header

tenant id of base policy

For Azure China, authentication endpoint need to updated as below

In the TrustFrameworkBase.xml, update the TechnicalProfile "login-NonInteractive":

```xml
<Item Key="ProviderName">https://sts.chinacloudapi.cn/</Item>
<Item Key="METADATA">https://login.chinacloudapi.cn/{tenant}/.well-known/openid-configuration</Item>
<Item Key="authorization_endpoint">https://login.chinacloudapi.cn/{tenant}/oauth2/token</Item>
```
