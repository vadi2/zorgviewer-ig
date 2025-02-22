
Op deze pagina volgen de checklists die als handvat kunnen worden gebruikt als je als bronsysteem wilt aansluiten (bouwblok Ontsluiten Bronsysteem) of als je de Zorgviewer wilt kunnen opstarten vanuit je eigen systeem (bouwblok Zorgviewer Host).
Maar ook voor het toevoegen van een nieuw behandelplan met relevante filters voor bepaalde aandoening of gewoon een setje relevante gegevens filters op gegevens.

### Basis afspraken

1. *Vulling*: Zorgaanbieder is verantwoordelijk om zoveel mogelijk van de Zorgviewer implementatiegids zib vulling te regelen
1. *Vulling*: Zorgaanbieder is er bewust van dat niet alle gegevens beschikbaar zijn in de Zorgviewer, of doordat het technisch (nog) niet mogelijk is (FHIR), of omdat de workflow niet is ingericht (Bronsysteem), of omdat configuratie niet volledig is (Bronsysteem of Bronsysteem ontsluiting), of omdat de Zorgviewer het betreffende veld (nog) niet toont.
1. *Vulling*: Zorgaanbieder accepteert dat vulling vanuit andere Zorgaanbieders minder kan zijn dan verwacht. We ontsluiten eerst wat we (technisch) kunnen. Bijvoorbeeld:
    1. codering mist, omdat coderingen niet (volledig) zijn ingelezen 
    1. regels missen, e.g. medische voorgeschiedenis
1. *Identiteit/Security*: Zorgaanbieder accepteert identiteit van de andere aangesloten Zorgaanbieders en mag ervanuit gaan dat er wordt gezorgd voor een juiste gebruikersrechten administratie.
1. *Security*: Zorgaanbieder staat in het NEN 7510 register
1. *Logging*: Zorgaanbieder moet voor de NEN 7513 conforme logging zorgen
1. *Governance*: Zorgaanbieder is deelnemer RIVO-Noord

### Epic huizen

<div>
{% include Checklist-Epic.svg %}
</div>

<style>table, td, th { border: 1px solid black; padding:5px; }</style>

**Stap 1 Aanmaken Apps in Epic: Zorgviewer FrontEnd en Zorgviewer Backend**<br/>
De frontend is een SMART-on-FHIR integratie, waarbij de standaard [Epic sequence er zo uit ziet](https://galaxy.epic.com/?#Browse/page=1!68!50!3404409,100001067,100001075,100001077&from=Galaxy-Redirect).
* 1.1 Front-end App: Interconnect (Foreground) verbinding kan maken naar https://auth-np.zorgviewer.nl/.well-known/jwks
* 1.2 Front-end App: Ga naar [My Apps / Vendor Services (epic.com)](https://vendorservices.epic.com/Developer/Apps) en maak de volgende App registratie:

| Veld | Vulling |
|--|--|
| Application Name:| `Zorgviewer-Frontend`|
| Who will primarily be using this app? | Clinicians, Staff, or Administrative Users|
| Features:| Incoming API|
| Selecteer de volgende Scopes:|* Patient.Read (STU3)<br/>* Practitioner.Read (STU3)|
| Does your app use OAuth 2.0? |Use Oauth 2.0|
| App FHIR Version: |STU3|
| FHIR ID Generation Scheme: |Use Unconstrained FHIR ID's|
| Endpoint URI:|* `dev.zorgviewer.nl/api/application/redirect`<br/>* `app-dev.zorgviewer.nl/api/application/redirect`<br/>* `app-tst.zorgviewer.nl/api/application/redirect`<br/>* `app-acc.zorgviewer.nl/api/application/redirect`<br/>* `app.zorgviewer.nl/api/application/redirect`|
| Is this a confidential Client? | Disable|
| Advanced: | * Enable on Sandbox: Disable |
| Non-Production Client ID: | Activate for Non-Production (production volgt in een later stadium)|

* 1.3 Back-end App: Ga naar [My Apps / Vendor Services (epic.com)](https://vendorservices.epic.com/Developer/Apps) en maak de volgende App registratie:

| Veld | Vulling |
|--|--|
| Application Name: |`Zorgviewer-Backend`|
| Who will primarily be using this app? | Backend Systems|
| Features:| Incoming API|
| Selecteer de volgende Scopes:|* Binary.Read (Correspondences) (STU3)<br/>* Binary.Read (Radiology Results) (STU3)<br/>* Condition.Search (Encounter Diagnosis, Problems) (STU3)<br/>* Consent.Search (Code Status) (STU3)<br/>* Consent.Search (Document) (STU3)<br/>* DocumentReference.Search (Correspondences) (STU3)<br/>* DocumentReference.Search (Radiology Results) (STU3)<br/>* Observation.$lastn (Labs) (STU3)<br/>* Observation.Search (Labs) (STU3)<br/>* Patient.Search (STU3)<br/>* Procedure.Search (Orders, Surgeries) (STU3)<br/>* Specimen.Read (Labs) (STU3)|
| Does your app use OAuth 2.0? |Use Oauth 2.0|
| App FHIR Version:| STU3|
| FHIR ID Generation Scheme: |Use Unconstrained FIHR ID's|
| Non-Production JWK Set URL: | `https://auth-np.zorgviewer.nl/.well-known/jwks`|
| Production JWK Set URL: | voorlopig leeg laten|
| Advanced: |* Enable on Sandbox: Disable<br/>* Non-Production Client ID: Activate for Non-Production (production volgt in een later stadium)<br/>* Ter info: deze waarschuwing kan genegeerd worden "Add Non-Production Credentials"|

**Stap 2 Client Certificate van de Zorgviewer back-end**
* 2.1 Team Zorgviewer: Het Zorgviewer team genereert een Zorgviewer-Bronsysteem specifiek Certificate Request (Client Certificaat) en leveren dit aan het aan te sluiten organisatie.
* 2.2 Op basis van het door Zorgviewer gegenereerde CSR, vraag een Client Certificaat aan. Dit mag een Publieke CA zijn, maar mag ook uitgegeven zijn door een Interne CA. 
	* Deel het Certificaat met team Zorgviewer.
* 2.3 Team Zorgviewer: PFX genereren op basis van Private Key + Gegenereerde Certificaat en opnemen in de Zorgviewer KeyVault

**Stap 3 Parallel kan het volgende worden geregeld**
* 3.1 Maak een backend User (EMP) aan met de volgende security points:<br/>
	**! Let op: wanneer meer informatie (zibs) worden gedeeld, kan het zijn dat er aanvullende security points nodig zijn.**
	* EpicCare Ambulatory security point 1-Patient Search/Select
	* EpicCare Ambulatory security point 16-Chart Review
	* EpicCare Ambulatory security point 54-Demographics
	* EpicCare Ambulatory security point 94-Chart Review – Order Tabs
	* EpicCare Ambulatory security point 111-Problem List
	* EpicCare Ambulatory security point 176-Review Procedure Reports
	* EpicCare Ambulatory security point 262-View Only Demographics
	* EpicCare Ambulatory security point 311-Order Review
	* EpicCare Ambulatory security point 333-Results Review
	* EpicCare Inpatient security point 4-Results Review
	* EpicCare Inpatient security point 5-Patient Summary
	* EpicCare Inpatient security point 6-Demographics
	* EpicCare Inpatient security point 7-Chart Review
	* EpicCare Inpatient security point 8-View Only Demographics
	* EpicCare Inpatient security point 12-Order Review
	* EpicCare Inpatient security point 13-Problem List
	* EpicCare Inpatient security point 184-View Procedure Reports
	* Nurse Triage/Call Management security point 902-Chart Review
	* Nurse Triage/Call Management security point 903-Demographics
	* MyChart - Hyperspace User security point 47 – Third Party View Questionnaires
	* Care Everywhere security point 4-View Documents
	* Cadence security point 5105-Edit Patient Record
	* Cadence security point 5201-Open Patient Record
	* EpicCare security point 35-Create New Patient
	* Identity security point 1-Create Record
* 3.2 *Moet nog aangevuld worden...* - FHIR endpoint (interconnect) > client id koppelen aan emp (epic manual)
Er moet een backend EMP worden aangemaakt, zie hiervoor de  [Epic Galaxy documentatie Backend System Integration](https://galaxy.epic.com/Redirect.aspx?DocumentID=100001068&PrefDocID=97042)
* 3.3 Uitvoeren van de Epic Sherlock Checklist BgZ VIPP5 voor de zorginformatiebouwstenen in scope
* 3.4 Om te voorkomen dat onterecht een werkdiagnose (die wel in problemlijst mee komen) op "confirmed" worden gezet, moet de FHIR Profile Variabele (AIP) FHIR_NL_PROBLEM_SHOW_VERIFY_STATUS ingesteld worden op 0 (Nee)

**Stap 4: Configureren EndPoints**
* 4.1 (UMCG OPLOSSING) Patiënttoestemmingscheck bouwen in broker. De gekozen oplossing is afhankelijk van het eigen organisatie. In het UMCG is dit als volgt geimplementeerd:
<div>
{% include Consent-seq.svg %}
</div>

* 4.1.1 In Epic: Ga naar Documenttype administratie: mapping van het Toestemmingsformulier onder de DocType Group van Patiënttoestemming. Dit is ook afhankelijk van de specifieke inrichting en beleid van de eigen organisatie.
* 4.2 *Moet nog aangevuld worden...* - Interconnect config + interne url's
* 4.3 Ontsluiten van Frontend en Backend end-points via broker naar internet en vervolgens de Zorgviewer IP-reeks 20.160.37.56/31 in de ACL van de Firewall (etc) opnemen zodat de Zorgviewer kan communiceren (HTTPS) met de endpoints. Dit is ook afhankelijk van de eigen organisatie hoe de beveiliging ingeregeld dient te worden. Indien de organisatie met een IP filter wilt werken is bovenstaande nodig.
* 4.4 Endpoint ontsluiten
	* Client Certificaat controle op BackendEndpoint
* 4.5 Aanleveren aan Zorgviewer volgende gegevens:
	* Client ID's
	* Backend (base en token) Endpoints externe URL's backend
	* ISS URL frontend
* 4.6 Toevoegen meta.source (HL7 NL OID van de zorgaanbieder) bij iedere FHIR resource in de FHIR response Bundle
```json
{
"entry": [ {
  "resource": {
    "resourceType": "...",
	  "id": "...",
      "meta": {
        "extension": [ {
            "url": "http://hl7.org/fhir/R4/StructureDefinition/extension-Meta.source",
            "valueUri": "uri:oid:2.16.840.1.113883.2.4.3.8"
        } ]
    } }
  } ]
}
```

**Stap 5: Hyperspace configuratie Opstarten Zorgviewer**
* 5.1 FDI record maken voor de Zorgviewer. Gebruik Naming Convention van eigen organisatie.

| Veld | Vulling |
|--|--|
| Type: | PACS[1] |
| Model Record: | SMART ON FHIR |
| Patient ID Type: | MDN |
| Installation Mnemonic Values: | 1 URL: `https://dev.zorgviewer.nl/api/application/launch`<br/>2 Protected: `1`<br/>3 ClientID: eigen clientID<br/>4 Launchtype: 7 (of 6)<br/>7 Use edge browser: 1|

* 5.2 Knop (E2U) maken voor het kunnen opstarten van de Zorgviewer
	* 5.2.1 Plaats de knop in de patiëntencontext en respecteer hierbij eigen Break-the-Glass regels
	* 5.2.2 Knop is dan alleen beschikbaar wanneer iemand in een (poli)klinisch contact van een patiënt kan

**Stap 6: Testen van de volledige bouw**

### Chipsoft huizen

#### Ontsluiten bronsysteem 

1. Regel Zorgplatform contract, specifiek voor de BgZ en Documenten Services van de [Digital-Care API](https://developer.zorgplatform.online/digital-care)
1. OID's voor Zorgplatform ontsluiting en HiX Webintegratie activiteitendefinities
    * 2.16.840.1.113883.2.4.3.213.1.3 = Test Zorgviewer
    * 2.16.840.1.113883.2.4.3.213.1.2 = Acceptatie Zorgviewer
    * 2.16.840.1.113883.2.4.3.213.1.1 = Productie Zorgviewer
1. Eerste keer FHIR ontsluiting toepassen: Aanzetten / synchroniseren Zorgplatform

#### Opstarten van de Zorgviewer vanuit eigen EPD

1. Webintegratie configureren in HiX

### Topicus huizen met VIPLive

#### Ontsluiten bronsysteem 
1. ...

#### Opstarten van de Zorgviewer vanuit eigen EPD
1. ...

### Verbinding en TLS Certificaten

1. [NCSC ICT-beveiligingsrichtlijnen voor Transport Layer Security (TLS)](https://www.ncsc.nl/documenten/publicaties/2021/januari/19/ict-beveiligingsrichtlijnen-voor-transport-layer-security-2.1)
1. ?? Eisen?
1. ?? Waar kan/moet je de Client en Server aanvragen?
1. ...

### Behandelplan / zorgpad / zorgproces

1. Maak uitwerking volgend RadB analysesheet / ZiRA...
1. Installeer in Behandelplan bouwblok
