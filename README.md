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
China phone number has 11 digits, so claim "nationalNumber" needs to be updated as "RegularExpression="^[0-9]{1,11}$"".

```xml
 <ClaimType Id="nationalNumber">
   <DisplayName>Enter Phone Number</DisplayName>
   <DataType>string</DataType>
   <UserHelpText>Enter Phone Number</UserHelpText>
   <UserInputType>TextBox</UserInputType>
   <Restriction>
     <Pattern RegularExpression="^[0-9]{1,11}$" HelpText="" />
   </Restriction>
 </ClaimType>
```

## Link Social Account with Local Account

https://docs.microsoft.com/en-us/azure/active-directory-b2c/social-transformations

https://stackoverflow.com/questions/57109641/linking-multiple-social-accounts-to-azur-b2c-local-account-through-custom-polici

https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/main/SocialAccounts/TrustFrameworkBase.xml#L458


## Multi-tenant Sign In in China B2C

```xml
<ClaimsProvider>
        <Domain>MultiTenant-AAD</Domain>
        <DisplayName>Common AAD</DisplayName>
        <TechnicalProfiles>
            <TechnicalProfile Id="MultiTenant-AADCommon-OpenIdConnect">
            <DisplayName>Multi-Tenant AAD</DisplayName>
            <Description>Login with your mcpod/aadfederation account</Description>
            <Protocol Name="OpenIdConnect"/>
            <Metadata>
                <Item Key="METADATA">https://login.chinacloudapi.cn/common/.well-known/openid-configuration</Item>
                <!-- Update the Client ID below to the Application ID -->
                <Item Key="client_id">96ff1843-xxxx-xxxx-xxxx-aac7c51c9f22</Item>
                <Item Key="response_types">code</Item>
                <Item Key="scope">openid profile</Item>
                <Item Key="response_mode">form_post</Item>
                <Item Key="HttpBinding">POST</Item>
                <Item Key="UsePolicyInRedirectUri">false</Item>
                <Item Key="DiscoverMetadataByTokenIssuer">true</Item>
                <!-- The key below allows you to specify each of the Azure AD tenants that can be used to sign in. Update the GUIDs below for each tenant. -->
                <Item Key="ValidTokenIssuerPrefixes">https://sts.chinacloudapi.cn/954ddad8-xxxx-xxxx-xxxx-1316152d9587,https://sts.chinacloudapi.cn/97195526-xxxx-xxxx-xxxx-36faa3980a03</Item>
                <!-- The commented key below specifies that users from any tenant can sign-in. Uncomment if you would like anyone with an Azure AD account to be able to sign in. -->
                <!-- <Item Key="ValidTokenIssuerPrefixes">https://sts.chinacloudapi.cn/</Item> -->
            </Metadata>
            <CryptographicKeys>
                <Key Id="client_secret" StorageReferenceId="B2C_1A_AADAppMultiTenantSecret"/>
            </CryptographicKeys>
            <OutputClaims>
                <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid"/>
                <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
                <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
                <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
                <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
                <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
            </OutputClaims>
            <OutputClaimsTransformations>
                <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
                <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
                <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
                <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
            </OutputClaimsTransformations>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin"/>
            </TechnicalProfile>
        </TechnicalProfiles>
    </ClaimsProvider>
```
