# B2C Custom Policy

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

| Orchestration Step  | Technical Policy |
| :------------ |:---------------:|
| 2 | LocalAccountSignUpWithLogonEmail |
| 3 | AAD-UserReadUsingObjectId |
| 4 | JwtIssuer     |

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

## Local Account SignIn With MFA

Require MFA after user sign-in, enroll and validate phone number.

### Official Sample

https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/main/SocialAndLocalAccountsWithMfa/TrustFrameworkBase.xml

### Doc for Phone Factor

https://docs.microsoft.com/en-us/azure/active-directory-b2c/phone-factor-technical-profile

## Phone SignIn and SignUp (passwordless)

### Official Sample

https://github.com/azure-ad-b2c/samples/tree/master/policies/signup-signin-with-phone-number

### Tips for policy editing

It has to do with the order that these children appear in BuildingBlocks, referring to the order listed here: https://docs.microsoft.com/en-us/azure/active-directory-b2c/buildingblocks. Otherwise, there would be errors like "The element 'BuildingBlocks' in namespace 'http://schemas.microsoft.com/online/cpim/schemas/2013/06' has invalid child element 'Predicates' in namespace 'http://schemas.microsoft.com/online/cpim/schemas/2013/06'. "

```xml
<BuildingBlocks>
    <ClaimsSchema>...</ClaimsSchema>
    <Predicates>...</Predicates>
    <PredicateValidations>...</PredicateValidations>
    <ClaimsTransformations>...</ClaimsTransformations>
    <ContentDefinitions>...</ContentDefinitions>
    <Localization>...</Localization>
    <DisplayControls>...</DisplayControls>
    <InputValidations>...</InputValidations>
</BuildingBlocks>
```
#### How to setup default selection

```xml
<ClaimType Id="countryCode">
     <DisplayName>Phone Number</DisplayName>
     <DataType>string</DataType>
     <UserHelpText>Phone Number</UserHelpText>
     <UserInputType>DropdownSingleSelect</UserInputType>
     <Restriction>
          ...
          <Enumeration Text="China(+86)" Value="CN" SelectByDefault="true"/>
          ...
     </Restriction>
</ClaimType>
```

## Link Social Account with Local Account

https://docs.microsoft.com/en-us/azure/active-directory-b2c/social-transformations

https://stackoverflow.com/questions/57109641/linking-multiple-social-accounts-to-azur-b2c-local-account-through-custom-polici

https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/main/SocialAccounts/TrustFrameworkBase.xml#L458
