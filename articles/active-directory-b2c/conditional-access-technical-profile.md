---
title: Conditional Access technical profiles in custom policies
titleSuffix: Azure AD B2C
description: Custom policy reference for Conditional Access technical profiles in Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg

ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/14/2020
ms.author: mimart
ms.subservice: B2C
---

# Define a Conditional Access technical profile in an Azure Active Directory B2C custom policy

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory (Azure AD) Conditional Access is the tool used by Azure AD B2C to bring signals together, make decisions, and enforce organizational policies. Automating risk assessment with policy conditions means risky sign-ins are at once identified and remediated or blocked.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## Protocol

The **Name** attribute of the **Protocol** element needs to be set to `Proprietary`. The **handler** attribute must contain the fully qualified name of the protocol handler assembly that is used by Azure AD B2C:

```
Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

The following example shows a Conditional Access technical profile:

```XML
<TechnicalProfile Id="ConditionalAccessEvaluation">
  <DisplayName>Conditional Access Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="OperationType">Evaluation</Item>
  </Metadata>
```

## Conditional Access evaluation

For every sign-in, Azure AD B2C evaluates all policies and ensures all requirements are met before granting the user access. "Block access" overrides all other configuration settings. The **Evaluation** mode of the Conditional Access technical profile evaluates the signals collected by Azure AD B2C during the sign-in with a local account. The outcome of the Conditional Access technical profile is a set of claims that result from Conditional Access evaluation. The Azure AD B2C policy uses these claims in a next orchestration step to take an action, such as block the user or challenge the user with multi-factor authentication. The following options can be configured for this mode.

### Metadata

| Attribute | Required | Description |
| --------- | -------- | ----------- |
| OperationType | Yes | Must be **Evaluation**.  |

### Input claims

The **InputClaims** element contains a list of claims to send to Conditional Access. You can also map the name of your claim to the name defined in the  Conditional Access technical profile.

| ClaimReferenceId | Required | Data Type | Description |
| --------- | -------- | ----------- |----------- |
| UserId | Yes | string | The identifier of the user who signs in. |
| AuthenticationMethodsUsed | Yes |stringCollection | The list of methods the user used to sign in. Possible values: `Password`, and `OneTimePasscode`. |
| IsFederated | Yes |boolean | Indicates whether or not a user signed in with a federated account. The value must be `false`. |
| IsMfaRegistered | Yes |boolean | Indicates whether the user already enrolled a phone number for multi-factor authentication. |


The **InputClaimsTransformations** element may contain a collection of **InputClaimsTransformation** elements that are used to modify the input claims or generate new ones before sending them to the Conditional Access service.

### Output claims

The **OutputClaims** element contains a list of claims generated by the ConditionalAccessProtocolProvider. You can also map the name of your claim to the name defined below.

| ClaimReferenceId | Required | Data Type | Description |
| --------- | -------- | ----------- |----------- |
| Challenges | Yes |stringCollection | List of actions to remediate the identified threat. Possible values: `block` |
| MultiConditionalAccessStatus | Yes | stringCollection |  |

The **OutputClaimsTransformations** element may contain a collection of **OutputClaimsTransformation** elements that are used to modify the output claims or generate new ones.

### Example: Evaluation

The following example shows a Conditional Access technical profile that is used to evaluate the sign-in threat.

```XML
<TechnicalProfile Id="ConditionalAccessEvaluation">
  <DisplayName>Conditional Access Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="OperationType">Evaluation</Item>
  </Metadata>
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="IsMfaRegisteredCT" />
  </InputClaimsTransformations>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="UserId" />
    <InputClaim ClaimTypeReferenceId="AuthenticationMethodsUsed" />
    <InputClaim ClaimTypeReferenceId="IsFederated" DefaultValue="false" />
    <InputClaim ClaimTypeReferenceId="IsMfaRegistered" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" PartnerClaimType="Challenges" />
    <OutputClaim ClaimTypeReferenceId="ConditionalAccessStatus" PartnerClaimType="MultiConditionalAccessStatus" />
  </OutputClaims>
</TechnicalProfile>
```

## Remediation

The **Remediation** mode of the Conditional Access technical profile informs Azure AD B2C that the sign-in identified threat is remediated. The following options can be configured for the remediation mode.

### Metadata

| Attribute | Required | Description |
| --------- | -------- | ----------- |
| OperationType | Yes | Must be **Remediation**.  |

### Input claims

The **InputClaims** element contains a list of claims to send to Conditional Access. You can also map the name of your claim to the name defined in the Conditional Access technical profile.

| ClaimReferenceId | Required | Data Type | Description |
| --------- | -------- | ----------- |----------- |
| ChallengesSatisfied | Yes | stringCollection| The list of satisfied challenges to remediate the identified threat as return from the evaluation mode, challenges claim.|


The **InputClaimsTransformations** element may contain a collection of **InputClaimsTransformation** elements that are used to modify the input claims or generate new ones before calling the Conditional Access service.

### Output claims

The Conditional Access protocol provider doesn't return any **OutputClaims**, so there's no need to specify output claims. You can, however, include claims that aren't returned by the Conditional Access protocol provider as long as you set the `DefaultValue` attribute.

The **OutputClaimsTransformations** element may contain a collection of **OutputClaimsTransformation** elements that are used to modify the output claims or generate new ones.


### Example: Remediation

The following example shows a Conditional Access technical profile used to remediate the identified threat:

```XML
<TechnicalProfile Id="ConditionalAccessRemediation">
  <DisplayName>Conditional Access Remediation</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ConditionalAccessProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
  <Metadata>
    <Item Key="OperationType">Remediation</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" PartnerClaimType="ChallengesSatisfied" />
  </InputClaims>
</TechnicalProfile>
```

To get the example working, add the claims shown in the following example to the ClaimsSchema section in your extensions file:

```XML
<!-- Conditional Access claims  -->
   <ClaimType Id="conditionalAccessClaimCollection">
      <DisplayName>Conditional Access claims</DisplayName>
      <DataType>stringCollection</DataType>
      <AdminHelpText>The list of claims which are the result of CA check</AdminHelpText>
   </ClaimType>
   <ClaimType Id="ConditionalAccessStatus">
      <DisplayName>The status of CA evaluation</DisplayName>
      <DataType>stringCollection</DataType>
      <AdminHelpText>The status of CA evaluation</AdminHelpText>
    </ClaimType>
    <ClaimType Id="AuthenticationMethodsUsed">
      <DisplayName>Authentication methods used</DisplayName>
      <DataType>stringCollection</DataType>
      <AdminHelpText>The list of authentication methods used</AdminHelpText>
    </ClaimType>
    <ClaimType Id="AuthenticationMethodUsed">
      <DisplayName>Authentication method used</DisplayName>
      <DataType>string</DataType>
      <AdminHelpText>The authentication method used</AdminHelpText>
    </ClaimType>
    <ClaimType Id="IsFederated">
      <DisplayName>IsFederated</DisplayName>
      <DataType>boolean</DataType>
      <AdminHelpText>Is user authenticated via an external identity provider</AdminHelpText>
    </ClaimType>
    <ClaimType Id="IsMfaRegistered">
      <DisplayName>IsMfaRegistered</DisplayName>
      <DataType>boolean</DataType>
      <AdminHelpText>Is user registered for MFA</AdminHelpText>
    </ClaimType> 
    <ClaimType Id="CAChallengeIsMfa">
      <DisplayName>CAChallengeIsMfa</DisplayName>
      <DataType>boolean</DataType>
    </ClaimType>
    <ClaimType Id="CAChallengeIsChgPwd">
      <DisplayName>CAChallengeIsChgPwd</DisplayName>
      <DataType>boolean</DataType>
    </ClaimType>
    <ClaimType Id="CAChallengeIsBlock">
      <DisplayName>CAChallengeIsBlock</DisplayName>
      <DataType>boolean</DataType>
    </ClaimType>
    <ClaimType Id="responseMsg">
      <DisplayName>responseMsg</DisplayName>
      <DataType>string</DataType>
      <AdminHelpText>A claim responsible for holding response messages to send to the relying party</AdminHelpText>
      <UserHelpText>A claim responsible for holding response messages to send to the relying party</UserHelpText>
      <UserInputType>Paragraph</UserInputType>
    </ClaimType>
<!-- End of Conditional Access claims -->
```

Add the following claims transformations:

```xml
<!--Conditional Access Transformations-->
  <ClaimsTransformation Id="AddToAuthenticationMethodsUsed" TransformationMethod="AddItemToStringCollection">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="AuthenticationMethodUsed" TransformationClaimType="item" />
      <InputClaim ClaimTypeReferenceId="AuthenticationMethodsUsed" TransformationClaimType="collection" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="AuthenticationMethodsUsed" TransformationClaimType="collection" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="CreatePasswordAuthenticationMethodClaim" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value="Password" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="AuthenticationMethodUsed" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="CreateOneTimePasscodeAuthenticationMethodClaim" TransformationMethod="CreateStringClaim">
    <InputParameters>
      <InputParameter Id="value" DataType="string" Value="OneTimePasscode" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="AuthenticationMethodUsed" TransformationClaimType="createdClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="IsMfaRegisteredCT" TransformationMethod="DoesClaimExist">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="strongAuthenticationPhoneNumber" TransformationClaimType="inputClaim" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="IsMfaRegistered" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
  <ClaimsTransformation Id="SetCAChallengeIsMfa" TransformationMethod="StringCollectionContains">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" TransformationClaimType="inputClaim" />
    </InputClaims>
    <InputParameters>
      <InputParameter Id="item" DataType="string" Value="mfa" />
      <InputParameter Id="ignoreCase" DataType="string" Value="true" />
    </InputParameters>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="CAChallengeIsMfa" TransformationClaimType="outputClaim" />
    </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="SetCAChallengeIsChgPwd" TransformationMethod="StringCollectionContains">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" TransformationClaimType="inputClaim" />
      </InputClaims>
      <InputParameters>
        <InputParameter Id="item" DataType="string" Value="chg_pwd" />
        <InputParameter Id="ignoreCase" DataType="string" Value="true" />
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="CAChallengeIsChgPwd" TransformationClaimType="outputClaim" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="SetCAChallengeIsBlock" TransformationMethod="StringCollectionContains">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" TransformationClaimType="inputClaim" />
      </InputClaims>
      <InputParameters>
        <InputParameter Id="item" DataType="string" Value="block" />
        <InputParameter Id="ignoreCase" DataType="string" Value="true" />
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="CAChallengeIsBlock" TransformationClaimType="outputClaim" />
      </OutputClaims>
    </ClaimsTransformation>
```

Add the following technical profile:

```xml
<TechnicalProfile Id="GenerateCAClaimFlags">
  <DisplayName>GenerateCAClaimFlags</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.ClaimsTransformationProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="CAChallengeIsMfa" />
    <OutputClaim ClaimTypeReferenceId="CAChallengeIsChgPwd" />
    <OutputClaim ClaimTypeReferenceId="CAChallengeIsBlock" />
  </OutputClaims>
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="SetCAChallengeIsMfa" />
    <OutputClaimsTransformation ReferenceId="SetCAChallengeIsChgPwd" />
    <OutputClaimsTransformation ReferenceId="SetCAChallengeIsBlock" />
  </OutputClaimsTransformations>
</TechnicalProfile>

<!--This is a self-asserted technical profile that we will use to show a block screen-->
<TechnicalProfile Id="ShowBlockPage">
  <DisplayName>Show Block message</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ContentDefinitionReferenceId">api.selfasserted.profileupdate</Item>
    <Item Key="TokenLifeTimeInSeconds">3600</Item>
    <Item Key="AllowGenerationOfClaimsWithNullValues">true</Item>
    <Item Key="setting.showContinueButton">false</Item>
    <Item Key="setting.showCancelButton">false</Item>
  </Metadata>
  <CryptographicKeys>
    <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
  </CryptographicKeys>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="responseMsg" DefaultValue="The user is blocked due to conditional access check." />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" />
  </OutputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
  <EnabledForUserJourneys>Always</EnabledForUserJourneys>
</TechnicalProfile>

```

In your TrustFrameworkPolicy element, add these SubJourneys as shown in the following example:

```xml
  <SubJourneys>
    <SubJourney Id="ConditionalAccess_Evaluation" Type="Call">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="ConditionalAccessEvaluation" TechnicalProfileReferenceId="ConditionalAccessEvaluation" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>conditionalAccessClaimCollection</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="GenerateCAClaimFlags" TechnicalProfileReferenceId="GenerateCAClaimFlags" />
          </ClaimsExchanges>
        </OrchestrationStep>
      </OrchestrationSteps>
    </SubJourney>

    <SubJourney Id="ConditionalAccess_Remediation" Type="Call">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>conditionalAccessClaimCollection</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="ConditionalAccessRemediation" TechnicalProfileReferenceId="ConditionalAccessRemediation" />
          </ClaimsExchanges>
        </OrchestrationStep>
      </OrchestrationSteps>
    </SubJourney>
  </SubJourneys>

```

Add a user journey that uses the new claims, as shown in the following example:

```xml
  <UserJourneys>
    <UserJourney Id="SignUpOrSignInWithCA">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
          <ClaimsProviderSelections>
            <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmailExchange" />

          </ClaimsProviderSelections>
          <ClaimsExchanges>
            <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <!-- Check if the user has selected to sign in using one of the social providers -->
        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
              <Value>objectId</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!-- For social IDP authentication, attempt to find the user account in the directory. -->
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
              <Value>authenticationSource</Value>
              <Value>socialIdpAuthentication</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="4" Type="InvokeSubJourney">
          <JourneyList>
            <Candidate SubJourneyReferenceId="ConditionalAccess_Evaluation" />
          </JourneyList>
        </OrchestrationStep>

        <!--MFA based on Conditional Access-->
        <OrchestrationStep Order="5" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>CAChallengeIsMfa</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
              <Value>CAChallengeIsMfa</Value>
              <Value>False</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="PhoneFactor-Verify" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!--Save MFA phone number: The precondition verifies whether the user provided a new number in the previous step. If so, the phone number is stored in the directory for future authentication requests.-->
        <OrchestrationStep Order="6" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>newPhoneNumberEntered</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="AADUserWriteWithObjectId" TechnicalProfileReferenceId="AAD-UserWritePhoneNumberUsingObjectId" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <OrchestrationStep Order="7" Type="ClaimsExchange" >
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>CAChallengeIsBlock</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="false">
              <Value>CAChallengeIsBlock</Value>
              <Value>True</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="ShowBlockPage" TechnicalProfileReferenceId="ShowBlockPage" />
          </ClaimsExchanges>
        </OrchestrationStep>

        <!--If a user has reached this point, this means a remediation was applied-->
        <!--  You can add a precondition here to call remediation only if a Conditional Access challenge was issued-->
        <OrchestrationStep Order="8" Type="InvokeSubJourney">
          <JourneyList>
            <Candidate SubJourneyReferenceId="ConditionalAccess_Remediation" />
          </JourneyList>
        </OrchestrationStep>
        <OrchestrationStep Order="9" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
      <ClientDefinition ReferenceId="DefaultWeb" />
    </UserJourney>
  </UserJourneys>

```

The following is an example of a relying party file that references this UserJourney:

```xml
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                      xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
                      xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
                      PolicySchemaVersion="0.3.0.0" TenantId="contoso.onmicrosoft.com" 
                      PolicyId="ha-sam-signup_signin-CA" 
                      PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_signup_signin_CA"
                      DeploymentMode="Development"
                      UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
  <BasePolicy>
    <TenantId>contoso.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignInWithCA" />
    <UserJourneyBehaviors>
      <SingleSignOn Scope="Tenant" />
      <SessionExpiryType>Absolute</SessionExpiryType>
      <SessionExpiryInSeconds>604800</SessionExpiryInSeconds>
      <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="<add your app insights instrumentation key" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
    </UserJourneyBehaviors>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="signInName" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub" />
        <OutputClaim ClaimTypeReferenceId="CAChallengeIsMfa" />
        <OutputClaim ClaimTypeReferenceId="CAChallengeIsBlock" />
        <OutputClaim ClaimTypeReferenceId="conditionalAccessClaimCollection" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

## Next steps

- You can find an example of a conditional access policy on [GitHub](https://github.com/azure-ad-b2c/samples/tree/master/policies/conditional-access).
