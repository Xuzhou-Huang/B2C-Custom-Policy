# B2C-Custom-Policy

## Modifications for Azure China
     
1. ReplyURL for App "IdentityExperienceFramework": https://xuzhoub2c.b2clogin.cn/xuzhoub2c.partner.onmschina.cn
2. Update "TenantId" and "PublicPolicyUri" in header of each policy (xml) file, as well as tenant id of base policy to CN b2c tenant name
3. For Azure China, authentication endpoint of local account (OIDC) need to updated as below. In the TrustFrameworkBase.xml, update the TechnicalProfile "login-NonInteractive":

```xml
<Item Key="ProviderName">https://sts.chinacloudapi.cn/</Item>
<Item Key="METADATA">https://login.chinacloudapi.cn/{tenant}/.well-known/openid-configuration</Item>
<Item Key="authorization_endpoint">https://login.chinacloudapi.cn/{tenant}/oauth2/token</Item>
```

## Local Account SignIn Or SignUp

### Official Sample

https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/main/LocalAccounts

### User Flow

#### User SignIn

| Orchestration Step  | Technical Policy |
| :------------ |:---------------:|
| 1 | SelfAsserted-LocalAccountSignin-Email |
| 3 | AAD-UserReadUsingObjectId |
| 4 | JwtIssuer     |

#### User SignUp

##### Hide SignUpLink

Add the following metadata in TechnicalProfile "SelfAsserted-LocalAccountSignin-Email" to hide the SignUp link:

```xml
 <Item Key="setting.showSignupLink">false</Item>
```

##### Set SignUpTarget

Metadata "SignUpTarget" of TechnicalProfile "SelfAsserted-LocalAccountSignin-Email" -- points to the Sign Up ClaimsExchange Id in a subsequent orchestrations step.
```xml
<Item Key="SignUpTarget">SignUpWithLogonEmailExchange</Item>
```
This enables the Sign Up link on the Combined Sign in and Sign up page to call the claims exchange in Orchestration Step 2, which consequently executes the LocalAccountSignUpWithLogonEmail technical profile.

| Orchestration Step  | Technical Policy |
| :------------ |:---------------:|
| 2 | LocalAccountSignUpWithLogonEmail |
| 3 | AAD-UserReadUsingObjectId |
| 4 | JwtIssuer     |

