<link rel="stylesheet" type="text/css" href="formatting.css" />

### Overview
This page is organized into several sections reflecting the various steps involved in implementing DTR.  These steps are organized in roughly the order they would typically occur, though some variation in order is possible.

The core of this process is summarized in this diagram:

[![DTRQuestionnairePackageOperation](questionnaire-package-sequence.png){:style="float: none;width:732px;height:559px"}](questionnaire-package-sequence.png "Sequence Diagram")

---------------------
### General Considerations
This implementation guide adopts by reference the following HRex Conformance Requirements which describe conformance language, rules around mustSupport and other considerations. 

#### Must Support
This IG marks elements with the Must Support flag in its profiles. In addition to the expectations provided in the HRex section referenced above, the following additional considerations apply:  

<!-- This guide profiles on top of US Core profiles where Must Support has been defined on certain elements (e.g., DTR Medication Request profile). --> The FHIR specification makes it clear that when profiling another profile, a MustSupport flag can be constrained further (i.e., taken from 'false' to 'true') but cannot be loosened (i.e., changed from 'true' to 'false').   
  
Da Vinci DTR implementations **SHALL** conform to the US Core IG [Must Support](http://hl7.org/fhir/us/core/STU3.1.1/general-guidance.html#must-support) Guidance where US Core IG resources are used.
  
DTR apps and EHRs that take on DTR app responsibility **SHALL** be able to render Questionnaires and QuestionnaireResponses.  Specifically, apps and EHRs acting as form fillers **SHALL** be able to display:
* `QuestionnaireResponse.questionnaire.questionnaireDisplay`/`Questionnaire.title`
* `QuestionnaireResponse.authored`
* `QuestionnaireResponse.author.resolve().name`
* `QuestionnaireResponse.source.resolve().name`
* `QuestionnaireResponse.item.text`
* `QuestionnaireResponse.item.value` - all data types
* The same for all nested items

They **SHALL** also handle all mustSupport elements within the Questionnaire profile and provide visual cues where those elements impact expected user action (e.g., required answers, need for signatures, etc.)
  
Those same systems **SHOULD** be able to display QR.item.itemMedia

#### EHRs and Payers
Throughout this guide, the term 'EHR' refers to the set of systems acting on behalf of a provider, whether managed directly by the provider organization or by a third-party agency.  Similarly, 'payer' refers to the set of systems acting on behalf of the payer, whether managed directly by the payer or by a third-party agency.  For additional discussion about how this implementation guide functions when EHR and/or payer systems are not monolithic but instead are made up of components, refer to the [ePA Coordinators page](epa.html).

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")  

---------------------
### Impact on payer processes
Information passed to DTR will typically contain clinical terminologies, might not contain billing terminologies, and will generally not include billing modifier codes or similar information typically included in prior authorization requests. Services will also need to consider that the mapping they perform between clinical terminologies and billing codes **MAY** be different than the bill coding process performed by the client system when claims are eventually submitted. This **MAY** mean that assertions about coverage or prior authorization requirements will need to be expressed conditionally. e.g., "Provided this service is billed as X, Y or Z, then prior authorization is not needed".

In situations where the DTR App is aware of the likely billing codes at the time of ordering, it might send these codes as additional CodeableConcept.coding repetitions to assist in server processing. If CPT is used, note the ability to convey CPT modifier codes via post-coordination as described in the Using CPT page on [terminology.hl7.org](https://terminology.hl7.org/).

It is more efficient if mappings can be shared across payers and providers. This implementation guide encourages industry participants to cooperate on the development of shared mappings and/or to work with terminology developers (e.g., AMA for CPT codes) to develop shared mappings as part of their code maintenance process.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Configuring EHR to App Connectivity
EHRs have the option of delivering DTR functionality internally or leveraging a DTR SMART Application that handles the primary data capture and population functions of the DTR process.  Such SMART applications **MAY** be developed by the EHR vendors, by payers, or by third parties.

DTR apps are expected to be technically able to run against any EHR and work with any payer. This means that EHRs are free to choose which app they prefer and **MAY** switch apps as they see fit.  However, for a DTR app to be used it needs to be trusted appropriately to manage access to personal health information by the EHRs and payers. EHRs that leverage DTR SMART apps will choose which DTR apps they will trust and support.  In some cases, an EHR **MAY** opt to support multiple DTR SMART apps.  If so, the EHR will be responsible for determining which app is used in which circumstances.

SMART on FHIR defines the [basic expectations for an EHR to register a SMART app](https://www.hl7.org/fhir/smart-app-launch/app-launch.html#register-app-with-ehr).  Most EHRs and organizations impose additional registration expectations.  DTR imposes an additional expectation as part of the app registration process, which is discussed in more detail in the [Determination of Payers Supported by a DTR App](specification.html#determination-of-payers-supported-by-a-dtr-app) section of this specification.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Configuring App/EHR to Payer Connectivity
Similarly, all DTR apps **SHALL** be registered with the payer systems with which they communicate.  If an EHR opts to interact with the payer directly without using an app, then the EHR itself will need to register.

Payers **MAY** have multiple back-end functions that handle different types of decision support and/or different types of services. However, for the purpose of DTR conformance, payers **SHALL** have a single endpoint (managed by themselves or a delegate) that can handle responding to all DTR service calls.

This registration process will ensure the that the DTR app or Full EHR:
* Is 'trusted' by the payer to deal with patient-identifiable data.
* Knows the relevant endpoint to use for the payer to access the [`$questionnaire-package`](OperationDefinition-questionnaire-package.html) operation has a shared secret allowing secure access to the payer endpoint via [SMART on FHIR Backend Services](https://www.hl7.org/fhir/smart-app-launch/backend-services.html).
  
Even after an application has been successfully registered, payers and EHRs **SHOULD** monitor the application behavior and **MAY** suspend an application's access if it is suspected of malicious behavior. 

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
Some payers may create Questionnaires that rely on code systems that require licensing for use where there isn’t already a cross-U.S. license in place (e.g., UB, CPT).  In such cases, the payer, provider, and if necessary, the SMART on FHIR app vendor will need to have the relevant licenses to display, store, and retransmit the information captured.  The expectations around licensing requirements should be established as part of the configuration process between the parties.
      </td></tr>
  </table>
</div><br>

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Defining Questionnaires

#### Questionnaire Use
Payers have documentation requirements and rules that must be satisfied as part of their typical operations when reimbursing for care for certain services requiring authorization or documentation.
The Questionnaire resource is used to represent the information needs that providers must satisfy to meet payer documentation requirements and/or rules. The Questionnaire resource allows specifying:  

* What questions are asked,
* What constraints exist on allowed answers (data types, choice lists, etc.), 
* Logic about which questions are enabled or disabled based on prior answers or other context,
* How questions are organized and grouped,
* Rules about what answers are mandatory or **MAY** have multiple answers,
* How the questions should be rendered (tables, drop-downs, etc.)
* How to automatically fill in answers to some of the questions based on data already available from the EHR.
* Etc.
  
The base Questionnaire resource defines some of these capabilities.  However, to allow specifying all the expectations needed to meet payer requirements for data collection forms, additional extensions are necessary.  DTR leverages the extended Questionnaire capabilities defined in the [Structured Data Capture (SDC) implementation guide](http://hl7.org/fhir/uv/sdc/STU3/) to define the complete set of functionality necessary to support data capture for payer purposes.  

More information regarding Questionnaires, workflow, and behaviors implementers will need to understand can be found in the SDC Guide:
* [Basic SDC Workflow](http://hl7.org/fhir/uv/sdc/STU3/workflow.html)
* [Form Behavior and Calculation](http://hl7.org/fhir/uv/sdc/STU3/behavior.html)
* [Advanced Form Rendering](http://hl7.org/fhir/uv/sdc/STU3/rendering.html)
* [DTR SDC Questionnaire - Example](Questionnaire-home-o2-std-questionnaire.html)
  
This IG defines two different profiles on Questionnaire that can be used to define data gathering requirements: the [DTR Standard Questionnaire profile](StructureDefinition-dtr-std-questionnaire.html) and the [DTR Adaptive Questionnaire profile](StructureDefinition-dtr-questionnaire-adapt.html).

In the first approach, all possible questions that can be asked, as well as the logic around when questions are enabled and what answers are permitted are expressed in computable form and shared as a complete set when the [questionnaire package](OperationDefinition-questionnaire-package.html) is retrieved.  Typically, the same Questionnaires are used across a wide variety of order types and members, though in theory the payer could generate a custom Questionnaire that is specific to a particular order and/or member.

In the second approach, the complete set of questions is not exposed.  Instead, a [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) operation is called repeatedly to slowly build up the list of questions, influenced by answers to prior questions.  The set of questions asked will typically vary from patient to patient and order to order.

A summary of the trade-offs between the two approaches can be seen in the table below:

<table border="1">
  <tr>
    <th>Standard Questionnaire</th>
    <th>Adaptive Questionnaire</th>    
  </tr>
  <tr>
    <td>All questions and logic shared when the questionnaire package is first downloaded. i.e., all potential questions, logic and rules are visible to the EHR or DTR app.
    </td>
    <td>Questions are exposed one at a time (or occasionally a couple at a time) and only relevant questions are exposed.
Rules and logic are hidden and only questions relevant to the current member/order are visible to the EHR or DTR app
 </td>   
  </tr>
  <tr>
    <td>Only <code>$questionnaire-package</code> operation is needed</td>
    <td>Payer must support both <code>$questionnaire-package</code> and <code>$next-question</code> operations</td>    
  </tr>
  <tr>
    <td>Logic for which questions are asked and which answers are available is encoded in the Questionnaire, possibly using CQL, and is executed by the DTR app/EHR.</td>
    <td>Logic for which questions are asked and which answers are available is written in software maintained by the payer in the language of their choice.  The DTR app/EHR simply exposes whatever questions and answer choices are returned by the operation</td>    
  </tr>
  <tr>
    <td>Payers will need to write CQL and/or leverage advanced Questionnaire logic elements for both rules as well as data pre-population</td>
    <td>Payers will need to write CQL for data pre-population only (though will need to express rules in some programming/configuration language of their choice)</td>    
  </tr>
  <tr>
    <td>Rules that require accessing other payer systems or logic not expressible in CQL cannot be expressed.</td>
    <td>Code defining rules for what questions to display, the text of the question, the answers available, etc. can depend on calls to other payer interfaces and can leverage logic of any needed level of complexity, including using third-party rules engines.</td>    
  </tr>
  <tr>
    <td>Any population from payer data must occur at the time the <code>$questionnaire-package</code> operation is called and cannot be dependent on data populated from the EHR or entered by the user.</td>
    <td>Answers to later questions can be pre-populated from payer data based on previously answered questions (whether answered by pre-population from the EHR or by users)</td>    
  </tr>
  <tr>
    <td>Questionnaires and associated logic can be shared in a standardized way across payers as part of industry standardization efforts to drive consistency in data collection</td>
    <td>Individual questions could theoretically be shared, but any special logic around which questions are asked or answers are available would not be easily shareable</td>    
  </tr>
  <tr>
    <td>Logic cannot leverage payer data except as present in pre-populated answers in the initial QuestionnaireResponse provided in the <code>$questionnaire-package</code> operation</td>
    <td><code>$next-question</code> logic is free to leverage data held by the payer about the member, plan, or other information
    </td>    
  </tr>
  <tr>
    <td>Outcomes of the Questionnaire process are only reflected in answers in the QuestionnaireResponse
    </td>
    <td>The QuestionnaireResponse returned by <code>$next-question</code> can contain extensions as well as answers – such as the <a href="https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html">coverage-information</a> extension (see below)
    </td>    
  </tr>
  <tr>
    <td>The payer has no access to what answers have been provided in the QuestionnaireResponse until/unless the QuestionnaireResponse is later transmitted to the payer (e.g., as part of a prior authorization request or a claim).  The payer cannot inject any identifiers around commitments with respect to coverage or the satisfaction of authorization requirements.
    </td>
    <td>The payer receives the data each time a question is asked and, [while limited in how they can use that data], they can associate it with identifiers that allow them to adhere to commitments with respect to coverage or the satisfaction of authorization requirements.</td>    
  </tr>
</table><br>
  
Payers **MAY** support either approach, or **MAY** opt to provide some Questionnaires using one approach and others using the second based on the requirements of the form.  DTR apps and Full EHRs **SHALL** support both types of Questionnaires.

#### Adaptive Form Considerations
When a payer uses an Adaptive Form, they **SHALL** return a questionnaire instance compliant with the DTR [AdaptiveQuestionnaire-Search](StructureDefinition-dtr-questionnaire-adapt-search.html) profile. This will include a [`questionnaire-adaptive`](http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-questionnaireAdaptive) extension that indicates that the Questionnaire is adaptive and is also used to determine the endpoint on which the [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) operation should be called to start completing the QuestionnaireResponse.  If the extension is simply a boolean value of 'true', then the operation endpoint is the payer base URL.  (i.e., `[base]/$next-question`).  If the extension is a url, then that is the base for the next question (i.e., `[url]/$next-question`).  

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
If present, any questionnaire-adaptive url <b>SHALL</b> be a sub-url under the base for the payer and able to be accessed within the same SMART Backend Services connection as was established to make the <a href="OperationDefinition-questionnaire-package.html"><code>$questionnaire-package</code></a> call.
      </td></tr>
  </table>
</div><br>

The QuestionnaireResponse included in the Questionnaire package Bundle accompanying an adaptive Questionnaire will follow the convention of referencing a contained Questionnaire 'derivedFrom' the canonical for the Questionnaire being completed.  Typically, the QuestionnaireResponse and contained Questionnaire will contain no answers (or corresponding questions), though the payer **MAY** opt to include a few pre-populated answers for user review prior to soliciting additional questions using the [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) operation.

The [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) operation provides no opportunity to pass context to the server posing the questions.  It will have no information about who the member is, the nature of the order, or anything else.  Any information needed will need to be passed in as the answer to a question.  Payers **MAY** define pre-populatable questions to extract such information, using CQL to access the Questionnaire's launchContext or performing any necessary data retrieval.  The populated (and potentially user-reviewed or adjusted) answers can then be leveraged in subsequent calls to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) to determine what to ask next.

Adaptive questionnaires pose a slight challenge when it comes to preparing the Questionnaire package.  Unlike standard questionnaires, adaptive forms do not identify all their questions up-front.  As such, it is not clear exactly which ValueSets will be needed for coded elements, nor which Libraries might be needed to contain CQL to support pre-population.  There are three strategies payers can use – and payers **MAY** opt to combine strategies within a single Questionnaire:

* The payer **MAY** opt to include CQL Libraries and ValueSets in the package that are not actually referenced by any questions, on the prospect that they *might* be (or are likely to be) referenced by one of the questions at some point.  Libraries might also be included that define common helper functions that will be leveraged by question-specific CQL provided later.  The downside of this approach is that an adaptive form might draw from a library of hundreds of different questions but only actually use a handful.  This **MAY** result in needing to share an extremely large questionnaire package.  It **MAY** also expose more of the proprietary logic than the payer **MAY** wish to make known.  On the other hand, any CQL or code lists provided in the package is content that will not need to appear contained in the QuestionnaireResponse, which can lighten subsequent calls to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html). 
  
* The payer **MAY** add CQL Libraries and ValueSets as 'contained' resources inside the QuestionnaireResponse that are relevant to the questions that are part of the Questionnaire for each [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) call, slowly building up the set of resources that happen to be relevant to the questions actually asked.  The benefit of this approach is that only codes and CQL relevant to the questions asked needs to be shared.  The downside is that this content is then present on every call to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html).
  
* Finally, the payer can opt to specify the CQL and codes without using Libraries or ValueSets at all – the CQL can be sent in-line within the various Expression elements, and the codes can be listed directly as answerOption Codings.  This has similar advantages and disadvantages to the second option, though it has less overhead.  On the other hand, CQL and codes can't be shared across different items.

The [Privacy, Security, and Safety page](security.html) includes additional guidance on the use of Adaptive questionnaires.

#### Determinations from Adaptive Forms
In some cases, upon receiving enough answers from an adaptive form, a payer will be able to make assertions about coverage, prior authorization, and/or any 'additional documentation needed' similar to what is provided by the CRD process. This information needs to be made available to the DTR client in a computable fashion. To do so, the adaptive form service will place the coverage-information extension on the root of the [QuestionnaireResponse](StructureDefinition-dtr-questionnaireresponse.html), alongside the [context](StructureDefinition-qr-context.html) extension. 

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
It will be unusual for a coverage-information extension created by an adaptive form to come back saying 'additional documentation required', however there are theoretical use-cases for this to be useful and this specification does not prohibit such behavior. If this occurs, it <b>MAY</b> result in a subsequent launch of DTR, or could result in the DTR client prompting the user as to whether they want to move on to filling out the new form(s).
      </td></tr>
  </table>
</div><br>

DTR payers **SHALL** ONLY use DTR adaptive forms to return a coverage-information extension when:
* CRD has already been called and a coverage-information extension is already present on the relevant order; and
* Information is needed from a user that cannot be made available via the CRD process (either by pre-fetch or active query).

Payers **SHALL** return coverage-information as early in the burden reduction process as possible (i.e., use CRD in preference to DTR as much as possible).

If an adaptive questionnaire response includes an unsolicited determination that authorization requirements have been 'satisfied', the EHR **SHALL** allow the clinician to flag the provided determination number as "not valid".  For example, If they feel the determination was based on incorrect information. If a payer receives a new invocation of an adaptive form for the same order, they **SHALL** treat the result of the new completion as replacing any previous completion from a prior coverage determination process. 

#### Population
One of the core purposes of this specification is to automate the retrieval of documentation required by the payer in a manner that reduces provider burden. 

Pre-populating answers reduces burden in two ways:
* It reduces the time that providers or back-end staff need to spend looking through the record for information when completing forms; and
* It eliminates issues from being introduced as part of transcription and reduces the likelihood of relevant information being missed.
  
Pre-population requires that logic be embedded in the Questionnaire that identifies where in the EHR the information can be found.  This in turn requires that the information needed be reliably locatable.

Questionnaires **SHALL** include logic that supports population from the EHR where possible. Such logic **SHOULD** rely exclusively on data elements and search parameters defined either in US Core or HRex (including simple calculations there-on - e.g., age from birthdate). Translation between standard codes **SHOULD** be supported where possible.  Ideally, the design of questions in payer forms **SHOULD** consider what data is likely to be available for pre-population purposes, with an objective of minimizing provider data entry effort. 

Due to differences in workflows or information systems, clinical information **MAY** be represented in different FHIR resources or with different codes or code systems. Therefore, payer CQL **MAY** have to examine different resources or use value sets to find patient information. It is preferable to have more extensive CQL or value sets than require a user to input values that the rules were unable to find.

DTR uses the [SDC Expression-based Population mechanism](http://hl7.org/fhir/uv/sdc/STU3/populate.html#expression-based-population) as the mechanism for populating, and specifically the use of CQL as the expression language.  CQL **SHALL** be used when pre-populating elements using data gathered from the EHR.  This will mean using the launchContext, [variable](https://hl7.org/fhir/extensions/StructureDefinition-variable.html) and the [initialExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-initialExpression.html), [contextExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-contextExpression.html) or [calculatedExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-calculatedExpression.html) extensions together with logic in [Libraries](http://hl7.org/fhir/R4/library.html) referenced by the [cqf-library](https://hl7.org/fhir/extensions/StructureDefinition-cqf-library.html) extension to help populate the QuestionnaireResponse.

Further guidance on writing the necessary CQL can be found [later in this guide](specification.html#use-of-cql).

#### Questionnaire Design
Questionnaires, whether standard or adaptive, **SHOULD** also use logic that ensures that only 'relevant' questions and answer choices are displayed, based on what answers have already been provided/populated using elements such as enableWhen or enableWhenExpression. When using elements with a data type of  'Expression' within Questionnaires to control flow or rendering, all logic **SHALL** be written in CQL.  However, there are many other properties and extensions that control the rendering and behavior of Questionnaires.The DTR [Standard](StructureDefinition-dtr-std-questionnaire.html) and [Adaptive](StructureDefinition-dtr-questionnaire-adapt.html) Questionnaires have identified the elements and extensions from the SDC [Base](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire.html), [Advanced Rendering](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-render.html), [Advanced Behavior](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-behave.html) and [Expression-based Population](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-pop-exp.html) profiles that must be supported in DTR.  The descriptions Elements flagged as mustSupport **SHALL** be supported by DTR Apps and Full EHRs.  

These systems **SHOULD** also support all non mustSupport data extensions included in the differential of the DTR Questionnaire profiles as per SDC documentation for those elements and extensions and **SHALL** gracefully handle the presence of these elements if not supported. (i.e., non-support for an element **SHALL NOT** interfere with a user's ability to complete a QuestionnaireResponse.)  However, payers **SHALL NOT** rely on support for any of these elements in the design of their Questionnaire.  i.e., a DTR client that ignores such elements cannot impact the successful collection of information acceptability of the information gathered. 

The questionnaire designer should consider the possibility that access to some data **MAY** be restricted or prohibited and access restrictions **MAY** happen silently (e.g., indistinguishable absent data).

#### Provider Attestation
In some cases, if there isn't specific data that can be retrieved computably from the EHR, it **MAY** be sufficient for a payer to merely have an attestation by the provider that certain documentation exists, that a certain patient condition exists, or that certain actions have been completed. This can be represented in a Questionnaire as a simple boolean or choice question where the text describes what the user is attesting to. Payers **SHOULD** design questionnaires to support attestation rather than discrete data where this is sufficient for the business requirements.

Some payers **MAY** require that attestations or other answers be 'signed' (the electronic equivalent of 'initialing' the answer). This would be identified by means of the signatureRequired extension on the Questionnaire item.  

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
This IG does not define any expectations around certificate management or other considerations involved in the creation or validation of signatures, only the means by which signatures are requested and provided.
      </td></tr>
  </table>
</div><br>

Questionnaires **MAY** also support attaching reports or other supporting documentation (e.g., images, pathology reports, etc.) where providing question answers is not sufficient. The 'attachment' question type can be used to support this. Attachments might be found by searching for DiagnosticReport or Media instances, or by the provider directly uploading something to the Questionnaire rendering tool.

#### Questionnaire Expiry
While a user might need to suspend interaction with the DTR process, there could possibly be a limit on the amount of time that a set of documentation templates and rules is valid. For example, it is unreasonable to resume the DTR process for an order that was started several years ago because it would produce a QuestionnaireResponse that would still be valid if submitted now.  On the other hand, if a form was filled out five days ago and submitted today, it will be frustrating for providers if the form is rejected as "out of date".
Payers **SHOULD** use the Questionnaire.effectivePeriod element to describe the period over which the documentation templates and rules are valid. The DTR App or EHR can then check against that period when resuming a previously stored QuestionnaireResponse.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Value Set and Code System Guidance
The Questionnaire resource provides several mechanisms for conveying coded answer choices.  The table below is guidance that **SHOULD** be used when constructing questions with coded answers:

<table border="1">
  <tr>
    <th>CQL and Questionnaire terminology usage</th>
    <th>Comment</th>    
  </tr>
  <tr>
    <td>No value set, embed the code choices using answerOption.</td>
    <td>This is OK if there is no need to reuse the codes across multiple questions and the list of codes is ‘fixed’ and relatively small (< 40).</td>   
  </tr>
  <tr>
    <td>Pass a value set in the questionnaire package with the expansion already in place.</td>
    <td>The same set of codes is used for multiple answers and the set of codes is relatively small (< 40).  This saves the need for a terminology service call.’</td>    
  </tr>
  <tr>
    <td>Pass a value set in the questionnaire package, but the client recipient will need to run the expansion or ask a terminology server to do the expansion.</td>
    <td>The value set expansion is on the larger size (> ~40 codes), such that using the $expand with a filter operation will be more efficient from a user interface perspective.</td>    
  </tr>
</table><br>

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Use of CQL
As part of the SDC Questionnaires that define the information payers require to be captured, CQL is used to support automatic population of answers from existing EHR data and, occasionally, to enforce complex logic around what questions should be displayed. For example, whether answers are required, what answers should be enabled, etc. This CQL **MAY** query for patient observations, conditions, or other discrete information within the EHR to use as part of the population process or logic. The SDC specification provides guidance about how CQL expressions can be used for different purposes, as well as how information gathered by CQL in one portion of the Questionnaire can be made available in other portions of the Questionnaire.

CQL can either be embedded inline as part of an expression or referenced in a Library resource. All libraries needed by a questionnaire **SHALL** be referenced by the cqf-library extension and included as part of the [`$questionnaire-package`](OperationDefinition-questionnaire-package.html) operation. 

#### Guidance on Structure of CQL Logic
Like many other programming languages, CQL allows for statements to be nested within conditional logic. This creates instances where some statements **MAY** not be executed due to a prior condition not being met.  

Data retrieval in a QuestionnaireResponse is dependent on the `enableWhen` element.

CQL logic **SHOULD** be partitioned to be specific to groups/questions/etc. when doing so will allow it to be more efficient - though consideration should also be given to whether performing significant data gathering at the outset (even if the data is unneeded) will produce a more positive experience than intermittent data retrieval 'on demand', when such retrieval **MAY** introduce user-interface delays.
This pattern of logic structure is referred to by several names, including eager quitting, early return or, short circuiting. The goal is to avoid the execution of statements if they will not be relevant given other information available to the logic. This is done to streamline workflow and allow the user to focus on relevant input fields.

As an example, a payer **MAY** have a set of rules or specific information that must be gathered on a patient only if they have diabetes. This information **MAY** be gathered through a series of CQL statements. When constructing this CQL for DTR, these statements **SHOULD** be nested in conditionals to first check if the patient has diabetes before checking for information dependent on that condition.
  
<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
Implementers could use Adaptive Forms to minimize the need for any CQL that provides conditional information retrieval.
      </td></tr>
  </table>
</div><br>

#### Organizing CQL within Questionnaires
CQL can either be embedded inline in Expression elements or packaged in libraries.  This guide strongly recommends the practice of putting most CQL logic in libraries as it is easier to edit and debug such logic when is all in one place than when it's scattered through **MAY** different expression elements throughout the Questionnaire.
In this approach and results of the queries, filtering, and other operations are to variables in the Libraries using define statements. These variables can then be referenced in the [valueExpression](https://hl7.org/fhir/extensions/StructureDefinition-variable.html), [contextExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-contextExpression.html) or [calculatedExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-calculatedExpression.html) elements asserted for the various Questionnaire item elements.  If a given variable evaluates to null the answer will be left empty, for the end user to complete.

#### CQL Constraints
DTR imposes the following additional rules on CQL use by payers to maximize interoperability:
* The CQL **SHALL** be version [CQL 1.5](http://cql.hl7.org/N1/).
* CQL **SHALL** have a context of "Patient". 
* Within the Questionnaire, CQL **SHALL** follow SDC rules for determining context. Specifically, CQL definitions and variables defined on ancestor elements or preceding expression extensions within the same Questionnaire item are in scope for referencing in descendant/following expressions. 
* Within Libraries, both raw CQL and compiled ELM (in JSON syntax – i.e., application/elm+json) **SHALL** be provided as separate content repetitions within the library.  Within Expression elements, the base expression CQL **SHALL** be accompanied by a [US Public Health Alternative Expression Extension](http://hl7.org/fhir/us/ecr/StructureDefinition/us-ph-alternative-expression-extension) containing the compiled JSON ELM for the expression.
* If the Questionnaire depends on multiple Libraries (has multiple cqf-library elements), then any [valueExpression](https://hl7.org/fhir/extensions/StructureDefinition-variable.html) referring to defined variables **SHALL** specify the library name as well as the statement name as follows: "LibraryName".statementName.
* Library names **SHALL** be unique within a Questionnaire package and **SHOULD** be unique across all Libraries made available by the payer. e.g., "expression": "\"LowerLimbProsthesis\".PhysicalExaminationType"
where LowerLimbProsthesis is the library name and PhysicalExaminationType is the expression name.
* FHIR Libraries **SHALL** send CQL and ELM using the `content.data` element.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Determination of Payers Supported by a DTR App
It is possible that the apps used to provide DTR functionality will not support all payers the EHR might have "DTR requests" for. It is important for the EHR to know what payers their app supports so that they only allow their users to launch the DTR app in the context of payers the app will be able to support. (Launching an app only to be told "this payer isn't supported" is an unpleasant user experience.)  

The developer of SMART on FHIR DTR apps **SHALL** define an endpoint maintaining a list of payers currently supported by that app.  EHRs using external DTR apps **SHALL** support accessing the endpoint. The EHR will be configured with knowledge of which endpoint to hit for a given app as part of the process of configuring support for that app within the EHR. Different endpoints **SHALL** be defined for different versions of the application in situations where support for payers varies by application version.

Accessing the endpoint will be by a simple GET with an Accept header of application/json and **SHALL** be performed over TLS. The returned JSON object will contain a "payers" property referring to an array of objects. Each object will have an id and name property, both with string values. It is possible that additional properties **MAY** be supported in the future.

EHRs will typically retrieve the list of supported payers for the app once per day and will use this information to determine whether to expose the ability to launch DTR for orders associated with coverages for that payer.

Standardization of payer ids is still an open issue.  Guidance on standardization of payer identity is expected to be provided in the [CRD/DTR Supplemental Guide](https://confluence.hl7.org/pages/viewpage.action?pageId=91991946) on HL7 Confluence.
 
[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Launching DTR

#### Launching a DTR SMART app
If an EHR is not using a SMART app, this section will not apply.

When the DTR process is being launched, the Electronic Health Record (EHR) system and DTR process **SHALL** follow the procedures established by the SMART App Launch Framework - specifically the [SMART App Launch Framework's EHR launch sequence](https://www.hl7.org/fhir/smart-app-launch/app-launch.html#launch-app-ehr-launch).

Like any SMART app, DTR is passed a context when launched.  The openId, user, and patient launch contexts **SHALL** be requested and provided.  In addition, the launchContext **SHOULD** include fhirContext references as follows:
* One active Coverage resource indicating the coverage(s) associated with the documentation to be captured (unless Coverage is already conveyed in the [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extensions of the Request or Encounter passed below) and 
* Exactly one of the following:
  * A CRD-type Request or Encounter resource ([CRD profiles](https://build.fhir.org/ig/HL7/davinci-crd/artifacts.html#structures-resource-profiles))
    * Used when launching based on recommendations from CRD or for stand-alone launch without CRD.
  * An existing incomplete QuestionnaireResponse previously created with DTR.
    * Used when re-launching DTR to continue a previous session
  * A Questionnaire [Task](https://hl7.org/fhir/us/davinci-cdex/StructureDefinition-cdex-task-attachment-request.html)
    * Used when launching based on a request from [CDex](https://hl7.org/fhir/us/davinci-cdex/index.html), possibly prompted by instructions in PAS.
  
If these are not passed in as part of context, then the app **MAY** either raise an error or guide the user to select the needed records.

In addition to launch contexts, the SMART app will need to request scopes for the type of information it needs to access and manipulate.  Payer-provided Questionnaires **MAY** require access to a wide range of resources.  At a minimum, the app will require read access to the following resources: 
> * Patient, 
> * Coverage, 
> * QuestionnaireResponse, 
> * Encounter, 
> * ServiceRequest, 
> * MedicationRequest, 
> * DeviceRequest, 
> * Appointment, 
> * NutritionOrder and 
> * VisionPrescription

Along with referenced resources such as :
> * Practitioner, 
> * PractitionerRole, 
> * Organization and 
> * Location.  

However, typically CQL in Questionnaires will also need access to Observation, Condition, Procedure, DocumentReference and potentially others.  Therefore, in most cases, apps will simply request `patient/*.rs` and `patient/questionnaireresponse.cu`.  However, apps **MAY** opt to be more restrictive in their access requests if they are confident that they can do so while meeting payer CQL needs and EHRs indicate this is desirable.

Regardless of the scopes asked for and granted by the user, the EHR **SHALL** limit access to only that data they deem appropriate for use in automatically populating payer forms, in particular restricting sensitive data.  (Refer to the [Privacy and Security section for more details](security.html))

#### Authenticating DTR client to payer API
To collect data for a payer, the EHR or SMART app responsible for data collection must first establish a connection to the payer.   Payers **SHALL** require DTR apps and EHRs connecting to their endpoint to authenticate.  using SMART on FHIR Backend Services.  This mechanism provides a flow that authorizes a system to connect to a FHIR server where no user needs to be involved in the authorization process. It allows systems like EHRs to easily interact with pre-authorized defined scopes of access. The scopes necessary are:
* `system/questionnaire.rs`
* `system/valueset.rs`
* `system/library.rs`

These are sufficient to invoke the DTR Questionnaire operations [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) and ValueSet/`$expand`, as well as retrieval of libraries and value sets.


#### Retrieving Launch Context Information
Again, this is only relevant for SMART on FHIR clients, though a similar process will happen for Full EHRs.
There are several steps in processing the launch context to begin the data collection process:
1.	If the launch context did not include Coverages, a QuestionnaireResponse or a Request or Encounter, the DTR app **MAY** error, but **SHOULD** allow the user to search to find one to use as context for the DTR session.  QuestionnaireResponses would be filtered by patient, a status of in-progress, and a context that is not empty, while requests and encounters would be filtered by patient, active status, and any other standard search parameters the DTR client chooses to offer.
2.	Otherwise, retrieve the referenced fhirUser, patient, and fhirContext resources referenced in the SMART authorization response by performing read or search operations.  (Search operations allow _including the requester, performer and location resources for encounters and request resources, which will be needed later.)
3.	For any Request resources or Encounter resources passed in context, perform a search to see if there are any QuestionnaireResponses associated with those orders using the [dtr-context](SearchParameter-dtr-context.html) search parameter.
e.g., `[EHR-base]/Questionnaire?patient=123&dtr-context=ServiceRequest/789`
4.	If no Request or Encounter resource was passed as context, check any provided QuestionnaireResponse for a 'context' extension identifying any associated requests or encounters, and if so, perform searches to retrieve them with _includes for requester, performer, etc.
5.	Look in the Request or Encounter resources (if any) passed from context or referenced by a QuestionnaireResponse (if any) to see if there is an existing [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extension, either from CRD or from DTR.
6.	If found, look at the Coverages as well as the coverage-assertion-id in such extensions.
7.	Determine which payers are associated with the coverages found, and then retrieve Questionnaire packages from each payer using the context information found.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Retrieving Questionnaire Packages
Once the EHR or app has authenticated itself, it can invoke the [`$questionnaire-package`](OperationDefinition-questionnaire-package.html) operation using the information gathered in the preceding [Retrieving Launch Context Information](specification.html#retrieving-launch-context-information) step.  The operation will be invoked with a 'POST' using the base endpoint associated with the coverages provided based on payer configuration.  If multiple Coverages are provided that are associated with payers having different endpoints, then the operation will be invoked once per payer endpoint.
e.g., `POST [base]/$questionnaire-package`

The operation will be invoked passing a [DTR Questionnaire Package Input Parameters](StructureDefinition-dtr-qpackage-input-parameters.html) instance containing the following information:
* All Coverages associated with the payer endpoint in repetitions of 'coverage'
* If a Questionnaire was identified, the version-specific canonical of that Questionnaire in 'questionnaire'.
* If no Questionnaires were identified but Request or Encounter resources were found in the preceding step, those resources as the 'order' resources, plus the associated performer and requester Practitioner and Organizations, as well as any referenced Devices or Medications as 'referenced' resources.
* If a coverage-information was available, the coverage-assertion-id as 'context'.
* Otherwise, the context id provided in the Questionnaire Task as 'context'.
* If working from a previously stored QuestionnaireResponse, the meta.lastUpdated as 'changedsince'.

The result will be a response with one or more Questionnaire package Bundles.  The remainder of the DTR process will involve the completion, storage, and use of the QuestionnaireResponses found in those Bundles.

If any of the retrieved Questionnaires have an effectivePeriod that ends prior to the current date, then the DTR client **SHALL** change the status of any retrieved work-in-progress QuestionnaireResponses for the expired Questionnaires to 'stopped' and notify the user that the previously recorded content has expired.  The [`$questionnaire-package`](OperationDefinition-questionnaire-package.html) operation would then need to be re-invoked just passing the orders and not specifying canonical questionnaire URLs to retrieve the updated Questionnaire(s) appropriate for the order, using the same 'context'.

If an expired Questionnaire is retrieved when it wasn't explicitly requested by referring to a canonical version, but instead by passing in the relevant order(s) and/or context id, the DTR client **SHOULD** report an error to the payer.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Pre-populating QuestionnaireResponses
Prior to exposing the draft QuestionnaireResponse to the user for completion and/or review, the DTR client **SHALL** execute all CQL necessary to resolve the [initialExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-initialExpression.html), [candidateExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-candidateExpression.html) and [calculatedExpression](http://hl7.org/fhir/uv/sdc/STU3/StructureDefinition-sdc-questionnaire-calculatedExpression.html) extensions found in the Questionnaire for any enabled elements.

All items that are pre-populated (whether by the payer in the initial QuestionnaireResponse provided in the questionnaire package, or from data retrieved from the EHR) **SHALL** have their `origin.source` set to 'auto'.

#### Execution Sequence
The flow of execution of the CQL will be determined by the associated Questionnaire. The client will proceed through the Questionnaire, and for any question that is associated with the result of a CQL expression, that specific CQL statement will be executed. The DTR client **SHALL** use result caching so that results that are already available will be reused without requesting them again.

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
The answers to questions populated by an initialExpression or calculatedExpression might themselves be dependencies to determine whether a Questionnaire item is enabled or not, which <b>MAY</b> in turn influence whether additional pre-population is required.  DTR clients <b>SHALL</b> iterate as necessary until a steady state is reached.  If dependencies are such that a steady state cannot be reached (e.g., an item that is enabled causes a value to be set which causes a different item to be enabled, that then disables the first item…), then the Questionnaire <b>SHALL</b> be treated as erroneous and attempts at automatic population <b>SHALL</b> end, with the user being informed of that.
      </td></tr>
  </table>
</div><br>

#### Retrieval of patient FHIR resources to supply to CQL execution engine
The DTR client **SHALL** retrieve the FHIR resources specified in the dataRequirement section of a Library.  SMART apps will do this using the access token provided on launch.  The client can then pass these resources to the Clinical Quality Language (CQL) engine. For example, the below snippet is from a Library that contains a dataRequirement section. In this code snippet the resource data needed from the EHR is Condition.

```json
"dataRequirement": [
    {
      "type": "Condition",
      "codeFilter": [
        {
          "path": "code",
          "valueSet": "http://cts.nlm.nih.gov/fhir/ValueSet/2.16.840.1.113762.1.4.1219.25"
        }
      ]
    },
```

Depending on user permissions, the client **MAY** not have access to all the data. The app's CQL execution engine **SHOULD** constrain queries to reduce data retrieval overhead. (For example, if the CQL logic filters for medications meeting certain conditions, it is acceptable if the engine retrieves all medications for the patient, even if a more constrained query was possible).
It's possible not every CQL statement will be executed (for example some questions **MAY** only be enabled given certain answers to prior questions). To reduce data transfers and increase overall speed, data **MAY** be fetched as needed. However, the app's execution engine **MAY** be implemented using a different strategy (for example by doing bulk fetches before starting execution).

#### Populating adaptive questionnaires
An adaptive form handles population slightly differently from a 'standard' form because questions only become available one (or a few) at a time.  Some CQL libraries **MAY** themselves only become available once certain questions are displayed.  The population process must therefore happen after each call to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html), populating any newly available questions.  DTR clients **SHALL** support such an incremental population of adaptive QuestionnaireResponses.  For performance, the DTR application **SHOULD** save the results of execution of CQL prior to the [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) call.

#### Populating resumed QuestionnairesResponses
If DTR is launched using a previously saved QuestionnaireResponse, the DTR client **SHOULD** re-execute CQL to populate all empty elements, as well as those with an `origin.source` of 'auto'.  Any answers with an `origin.source` of 'override' or 'manual' **SHALL NOT** be changed, though if pre-population would have asserted a value for an answer with an `origin.source` of 'manual', the `origin.source` **SHALL** be changed to 'override'.
An adaptive form handles population slightly differently from a 'standard' form because questions only become available one (or a few) at a time.  Some CQL libraries **MAY** themselves only become available once certain questions are displayed.  The population process must therefore happen after each call to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html), populating any newly available questions.  DTR clients **SHALL** support such an incremental population of adaptive QuestionnaireResponses.  For performance, the DTR application **SHOULD** save the results of execution of CQL prior to the [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) call.

#### Library Dependencies
The FHIR Library containing/referencing a CQL logic file can reference other needed CQL files (e.g., helper libraries) using the relatedArtifact field and a RelatedArtifact with a type of depends-on. The engine **SHALL** make available to the execution context all such referenced CQL libraries. 

#### Behavior when receiving malformed CQL
If the CQL is malformed (is not syntactically correct) in any way, the app's execution engine **SHALL NOT** attempt any execution of the malformed CQL, the client **SHALL** log the error, and the user **SHALL** be notified with an appropriate message indicating that population did not occur and they **SHALL** be allowed to enter the information manually either now or at a later time. The app **SHOULD** log failures and ensure the maintainer of the CQL/Questionnaire package is notified (see the DTR [Log Questionnaire Errors operation](OperationDefinition-log-questionnaire-errors.html) for details).
In this case, the details property of the OperationOutcome **SHOULD** use MSG_BAD_SYNTAX to indicate syntactical errors. The destination of the OperationOutcome will be the endpoint the malformed CQL/Questionnaire was retrieved from.

It is a CQL failure if the CQL cannot be executed by the app's CQL engine in the DTR client.

#### Behavior when encountering execution errors
If any errors are encountered during execution, the app's engine **SHALL NOT** attempt any further execution, and the user **SHALL** be notified with an appropriate on-screen error message. The app **SHALL** log failures and ensure the maintainer of the CQL/Questionnaire package is notified using the [DTR Log Questionnaire Errors](OperationDefinition-log-questionnaire-errors.html) operation. The user should have the option to complete the Questionnaire if possible, despite the error.

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
A query for data that returns no results <b>SHALL NOT</b> be considered a failure.
      </td></tr>
  </table>
</div><br>

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### User Interaction with Questionnaires
While the goal of DTR is to automatically gather all the necessary information to satisfy documentation requirements without interrupting the user, this is not possible in all cases.
It is likely that at least some answers will not be able to be gleaned from the EHR, due to missing data, data that is not computable, or data that is not represented in a standardized way. Also, even where answers are determined automatically, users may wish to review them for accuracy and completeness. Therefore, the system acting as a form filler is responsible for displaying all 'enabled' questions, groups, and display items to the end user for completion and/or review.
 In some cases, the population process **MAY** populate all answers to the Questionnaire. The DTR client **SHALL** provide the ability, but NOT a requirement, for providers to review pre-populated answers prior to saving the resulting response for subsequent use within the EHR.

#### Questionnaire Rendering
DTR leverages a subset of extensions and capabilities defined by the SDC implementation guide to support control over rendering, flow logic, and population and calculation of answers. The DTR SDC Questionnaire profile and DTR Adaptive Questionnaire profile identify the set of core elements and extensions that must be supported by 'full' EHRs and DTR solutions in terms of rendering and processing Questionnaires and their associated responses - and the elements that payers can count on being supported in the Questionnaires they expose.

Two different profiles are used to support two different approaches to managing questionnaire logic and two different levels of engagement between the form filling interface and the payer:
* DTR Standard Questionnaire 
* DTR Adaptive Questionnaire
  
Implementers should review the advanced rendering, advanced behavior, population and adaptive forms portions of the SDC implementation guide, focusing on the elements and extensions included in the DTR profiles. Implementers should also be familiar with the documentation about the Questionnaire and QuestionnaireResponse resources from the core FHIR specification. Conformance with DTR requires conformance with the relevant portions of the SDC implementation guide".

All DTR applications **SHALL** support rendering according to the mustSupport elements in the DTR Questionnaire profile as well as executing all CQL found within Questionnaire extensions. CQL and FHIR Questionnaires **SHALL** be required even when DTR is implemented within a DTR Native App as opposed to a DTR SMART App.

#### Separating user provided information from CQL retrieved information
When a user fills in a value or changes an answer in a QuestionnaireResponse, the DTR client **SHALL** populate the [InformationOrigin extension](StructureDefinition-information-origin.html), setting the 'author' property to the current user, and setting 'source' to 'override' if the source was already 'override' or 'auto', or 'manual' otherwise.

#### QuestionnaireResponse validation
Each Questionnaire has a set of rules that determine whether a QuestionnaireResponse is allowed to be considered 'complete'.  For example, cardinality expectations on answers, restrictions on allowed values, etc.  These rules are defined both by core elements as well as by extensions.

The DTR client **SHALL** validate the QuestionnaireResponse on an ongoing basis as the user is reviewing and entering data.   The client **SHALL** visually flag any elements that require adjustment to meet validation rules.  

For Standard Questionnaires, when the QuestionnaireResponse is valid, the DTR client **SHALL** indicate that to the user and allow them to mark the QuestionnaireResponse as complete, though the user **MAY** opt to continue making changes or save the QuestionnaireResponse in draft form.  The DTR app **SHALL NOT** mark a Standard QuestionnaireResponse as 'complete' automatically.  

With an adaptive Questionnaire, there is an iteration of invoking [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html), populating if possible, and allowing user review/adjustment of answers.  The DTR client **SHALL NOT** allow the user to indicate they are ready for the next question/set of questions until the answers to the current QuestionnaireResponse pass validation rules.  If [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) is invoked with a QuestionnaireResponse the payer determines is invalid based on the rules in the contained Questionnaire, it **SHOULD** return an HTTP 400 error with an OperationOutcome indicating the circumstances where the QuestionnaireResponse is invalid.

Unlike Standard Questionnaires, with adaptive QuestionnaireResponses, the determination that a QuestionnaireResponse is complete is made by the payer.  The final call to [`$next-question`](http://hl7.org/fhir/uv/sdc/STU3/OperationDefinition-Questionnaire-next-question.html) will result in an echo back of the QuestionnaireResponse with no further questions provided in the contained Questionnaire, and with the QuestionnaireResponse set to 'complete'.  However, the QuestionnaireResponse **MAY** have a [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extension added reflecting the payer's coverage assessment based on the information gathered in the QuestionnaireResponse.

#### Additional considerations
If a questionnaire or question is marked as 'signatureRequired', then the DTR client will need to add a 'signature' extension element to the appropriate location in the QuestionnaireResponse.  The signature would attest to the QuestionnaireResponse as a whole, or to the specific item the signature appears on – excluding any signature extensions themselves.
In some cases, the answer to a question modified by a user **MAY** be the input to an expression driving other logic within the questionnaire.  DTR clients **SHALL** monitor for changes to dependent questionnaire answers and re-execute logic, adjusting calculatedValues, enabling elements, adjusting validation rules, etc. based on changes made by the user.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Storing QuestionnaireResponses
If an EHR system performs DTR functionality internally, it **MAY** save session information however it likes. The guidance below does NOT apply in this scenario.

At any point prior to completion the DTR client **SHALL** be able to save the 'in-progress' QuestionnaireResponse, either to ensure that work-in-progress is not lost, or to allow the user to close the session and then relaunch it later. 
When the QuestionnaireResponse has been marked as 'complete', the DTR client **SHALL** save the QuestionnaireResponse to the EHR using a create (POST) if the record had not previously been stored, or an update (PUT) if the record had already been saved at least once. 
The DTR process creates a QuestionnaireResponse resource through the course of normal operation. This resource **SHOULD** be saved to the patient record in the EHR system of the healthcare provider if supported. It **MAY** also be transmitted to the payer IT system.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")

---------------------
### Passing DTR Questionnaire Responses to Downstream Processes
The information gathered via DTR can be used for a variety of purposes:
* For inclusion as a prior authorization attachment (either via PAS, CDex, or via traditional attachment submission mechanisms).
* For inclusion as a claims attachment as part of an X12 submission.
* To accompany the order so the information is available to the performing system as per the FHIR Orders Exchange IG
* To be retained by the EHR in the event of a subsequent audit.
* Additional purposes not yet defined.
  
Once DTR has written a QuestionnaireResponse to the DTR Client and updated the QuestionnaireResponse.status element to complete, the DTR Client needs to understand what action(s) it should take with the collected information. This is accomplished via three extensions:
1. The context extension provides a reference to the Request resource(s) and/or Encounter that the QuestionnaireResponse relates to.
2. The intendedUse extension indicates how the EHR is to use the information with respect to the associated orders/records
3. The [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extension captures any coverage determination made by the payer with respect to the Request or encounter resources mentioned in the 'context' extension.
 
Upon storing a completed QuestionnaireResponse with a [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extension, the EHR **SHALL** propagate the [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) extension to all non-Coverage resources included on the 'context' extension on the QuestionnaireResponse.  If a [coverage-information](https://build.fhir.org/ig/HL7/davinci-crd/StructureDefinition-ext-coverage-information.html) already exists on the target resource with the same coverage-information.coverage, it **SHALL** be overridden.

In those cases where a QuestionnaireResponse is to be used to convey information either to the payer or to downstream service providers, the DTR Client **SHALL** 'wrap' the QuestionnaireResponse in a 'collection' Bundle complying with the [questionnaireResponseBundle](StructureDefinition-DTR-QRBundle.html) profile. This Bundle will include the QuestionnaireResponse entry as its initial entry and will then also include any resources that are referenced by the QuestionnaireResponse as answerReference that are not already contained within the QuestionnaireResponse. This ensures that all necessary information is delivered without a need for subsequent queries.

<div markdown="1" class="notebox">
  <table style="border: none; margin-bottom: 0px;">
    <tr><td style="width: 72px; border: none"><img src="Note.png" style="float: left; width:18px; height:18px; margin: 0px;">&nbsp;<b><span style="color:maroon;">NOTE:</span></b></td>
      <td style="border: none"> 
Only those resources directly referenced in the QuestionnaireResponse are included. If further references from a referenced resource are desired, the designer of the Questionnaire must ensure that these are also included as answers (possibly as hidden answers automatically populated by CQL within the Questionnaire).
      </td></tr>
  </table>
</div><br>

While multiple purposes are possible, the expectation is that all information in the QuestionnaireResponse will be used for that purpose - e.g., If a QuestionnaireResponse had a purpose of 'prior authorization' and 'order transmission', then the full QuestionnaireResponse bundle would be sent as a prior authorization attachment and as an attachment when the order is sent to the performer.

If there is a desire to send different content to different recipients, then distinct QuestionnaireResponses **SHALL** be used.
It is up to the DTR Client to determine the process by which attachments to prior auth requests, claims or orders are assembled - this could be done automatically, or with human review.

[![ToTop](PageTop.png){:style="float: none;"}](specification.html "Back to top of page")
