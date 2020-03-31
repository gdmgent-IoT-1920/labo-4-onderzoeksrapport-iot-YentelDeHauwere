# Onderwerp: CoAp
Het Constrained Application Protocol (CoAP) is een gespecialiseerd weboverdrachtsprotocol voor gebruik met constrained nodes en beperkte netwerken in het **Internet of Things**. Het protocol is ontworpen voor machine-to-machine (M2M) -applicaties zoals smart energy en building automation.

CoAP is ontworpen voor gebruik tussen apparaten op hetzelfde beperkte netwerk (bijv. Low-power, lossy-netwerken), tussen apparaten en algemene knooppunten op internet, en tussen apparaten op verschillende beperkte netwerken die beide zijn verbonden met een internet. CoAP wordt ook gebruikt via andere mechanismen, zoals sms op mobiele communicatienetwerken.

CoAP is een servicelaagprotocol dat bedoeld is voor gebruik in bronnen met beperkte bronnen, zoals draadloze sensornetwerkknooppunten . CoAP is ontworpen om gemakkelijk te vertalen naar HTTP voor vereenvoudigde integratie met het web, terwijl het ook voldoet aan gespecialiseerde vereisten zoals multicast- ondersteuning, zeer lage overhead en eenvoud. Multicast, lage overhead en eenvoud zijn uiterst belangrijk voor Internet of Things (IoT) en Machine-to-Machine (M2M) -apparaten, die vaak diep ingebed zijnen hebben veel minder geheugen en voeding dan traditionele internetapparaten. Daarom is efficiëntie erg belangrijk. CoAP kan worden uitgevoerd op de meeste apparaten die UDP of een UDP-analoog ondersteunen.

De Internet Engineering Task Force ( IETF ) Constrained RESTful Environments Working Group ( CoRE ) heeft het belangrijkste standaardisatiewerk voor dit protocol gedaan. Om het protocol geschikt te maken voor IoT- en M2M-applicaties zijn er verschillende nieuwe functies toegevoegd. De kern van het protocol is gespecificeerd in RFC 7252 ; belangrijke uitbreidingen bevinden zich in verschillende fasen van het normalisatieproces.

![alt text][overview]

[overview]: https://engineersgarag.wpengine.com/wp-content/uploads/2019/07/Overview-CoAP-Protocol-Constraint-Devices.jpg "Overview of CoAP Protocol for Constraint Devices"


## Berichtformaten

Het kleinste CoAP-bericht is 4 bytes lang als Token, Opties en Payload worden weggelaten. CoAP maakt gebruik van twee berichttypen, verzoeken en antwoorden, met behulp van een eenvoudig, binair, standaard headerformaat. De basiskop kan worden gevolgd door opties in een geoptimaliseerde Type-Lengte-Waarde-indeling. CoAP is standaard gebonden aan UDP en optioneel aan DTLS , wat een hoog niveau van communicatiebeveiliging biedt.

Alle bytes na de headers in het pakket worden beschouwd als de berichttekst. De lengte van de berichttekst wordt geïmpliceerd door de datagramlengte. Indien gebonden aan UDP, MOET het hele bericht binnen een enkel datagram passen. Bij gebruik met 6LoWPAN zoals gedefinieerd in RFC 4944 , MOETEN berichten in een enkel IEEE 802.15.4- frame passen om fragmentatie te minimaliseren.


| Offsets | Octet | 0 | 1                      | 2          | 3 |
|---------|-------|---|------------------------|------------|---|
| **Octet**   | **Bit**   | 0 1 2 3 4 5 6 7  | 8 9 10 11 12 13 14 15 | 16 17 18 19 20 21 22 23    | 24 25 26 27 28 29 30 31|
| **4**      | **32**    | VER, Type, Token Length  | Request/Response Code  | Message ID | Message ID 
| **8**       | **64**    |   Token (0 - 8 bytes) | Token (0 - 8 bytes) | Token (0 - 8 bytes)| Token (0 - 8 bytes) |
| **12**      | **96**    | Token (0 - 8 bytes)  | Token (0 - 8 bytes) | Token (0 - 8 bytes) | Token (0 - 8 bytes)  |
| **16**      | **128**   |  Options (If Available) |  Options (If Available) |Options (If Available) |  Options (If Available) |
| **20**      | **160**   | 1 1 1 1 1 1 1 1  | Payload (If Available) |Payload (If Available)|Payload (If Available) |

### Vaste koptekst CoAP: versie, type, tokenlengte, verzoek- / responscode en bericht-ID.

De eerste 4 bytes zijn verplicht in alle CoAP-datagrammen.
Deze velden kunnen eenvoudig uit deze 4 bytes in C worden gehaald via deze macro's:

	  #define COAP_HEADER_VERSION (data) ((0xC0 & data [0]) >> 6) 
	  #define COAP_HEADER_TYPE (data) ((0x30 & data [0]) >> 4) 
	  #define COAP_HEADER_TKL (data) ((0x0F & data [ 0]) >> 0) 
	  #define COAP_HEADER_CLASS (data) (((data [1] >> 5) & 0x07)) 
	  #define COAP_HEADER_CODE (data) (((data [1] >> 0) & 0x1F)) 
	  #define COAP_HEADER_MID (data) ((data [2] << 8) | (data [3]))

#### Versie (VER) (2 bits) 

Geeft het CoAP-versienummer aan.

#### Type (2 bits)
Dit beschrijft het berichttype van het datagram voor de twee berichttype-context van Request and Response.

-   Verzoek
    -   0: Bevestigbaar: dit bericht verwacht een bijbehorend bevestigingsbericht.
    -   1: niet te bevestigen: dit bericht verwacht geen bevestigingsbericht.
-   Reactie
    -   2: Bevestiging: dit bericht is een antwoord dat een bevestigbaar bericht bevestigt
    -   3: Reset: dit bericht geeft aan dat het een bericht heeft ontvangen, maar het niet kan verwerken.

#### Tokenlengte (4 bits) 
Geeft de lengte aan van het veld Token met variabele lengte dat 0-8 bytes lang kan zijn.

#### Request / Response Code (8 bits)

| 0 1 2 | 3 4 5 6 7 |
|---------|---------|
| Klasse   | Code   | 


De drie meest significante bits vormen een nummer dat bekend staat als de "klasse", wat analoog is aan de. De vijf minst significante bits vormen een code die meer details over het verzoek of antwoord communiceert. De volledige code wordt doorgaans in het formulier meegedeeld `class.code`.

U kunt de nieuwste CoAP-aanvraag- / responscodes vinden op, hoewel de onderstaande lijst enkele voorbeelden geeft:

0.  Methode: 0.XX
    
    0.  LEEG
    1.  KRIJGEN
    2.  POST
    3.  LEGGEN
    4.  VERWIJDEREN
    5.  FETCH
    6.  PATCH
    7.  iPATCH
    
1.  Succes: 2.XX
    
    1.  Gemaakt
    2.  Verwijderd
    3.  Geldig
    4.  Veranderd
    5.  Inhoud
    6.  Doorgaan met
    

2.  Clientfout: 4.XX
    
    0.  Foutief verzoek
    1.  Ongeautoriseerd
    2.  Slechte optie
    3.  Verboden
    4.  Niet gevonden
    5.  methode niet toegestaan
    6.  Niet acceptabel
    7.  Aanvraagentiteit onvolledig
    8.  Conflict
    9.  Voorwaarde is mislukt
    10.  Aanvraagentiteit te groot
    11.  Niet-ondersteund inhoudsformaat
    

4.  Serverfout: 5.XX
    
    0.  Interne Server Fout
    1.  Niet geïmplementeerd
    2.  Slechte gateway
    3.  Service onbeschikbaar
    4.  Time-out van gateway
    5.  Proxying wordt niet ondersteund
    
5.  Signaalcodes: 7.XX
    
    0.  Niet toegewezen
    1.  CSM
    2.  Ping
    3.  Pong
    4.  Vrijlating
    5.  Afbreken

![alt text][rr]

[rr]: https://devopedia.org/images/article/90/4524.1529647227.jpg  "Request Repsonse"




#### Bericht-ID (16 bits) 
Wordt gebruikt om berichtduplicatie te detecteren en om berichten van het type Acknowledgement / Reset aan berichten van het type Confirmable / Non-confirmable te koppelen.: Antwoordberichten hebben dezelfde bericht-ID als het verzoek.

### Token 

Optioneel veld waarvan de grootte wordt aangegeven door het veld Tokenlengte, waarvan de waarden worden gegenereerd door de klant. De server moet elke tokenwaarde herhalen zonder enige wijziging terug naar de client. Het is bedoeld voor gebruik als een client-lokale ID om extra context te bieden voor bepaalde gelijktijdige transacties.

### Optie

**Optie-indeling**

| Bitposities | Bitposities|
|---------|---------|
| **0 1 2 3** | **4 5 6 7** |
| Optie Delta | Optie Length|
| Option Delta Extended (None, 8bit, 16bits) | Option Delta Extended (None, 8bit, 16bits)|
| Option Length Extended (None, 8bit, 16bits) | Option Length Extended (None, 8bit, 16bits)|
| Option Value | Option Value|


**Optie Delta:**

-   0 tot 12: Voor delta tussen 0 tot 12: Kleine delta tussen de laatste optie-id en de gewenste optie-id
-   13: Voor delta van 13 tot 268: Option Delta Extended is 8bit, dat is de waarde van Option Delta minus 13
-   14: Voor delta van 269 tot 65.804: Option Delta Extended is 16bit, dat is de waarde van Option Delta minus 269
-   15: Gereserveerd voor Payload Marker, waarbij de opties Delta / lengte samen zijn ingesteld als 0xFF.

**Optie lengte:**

-   0 tot 12: voor optielengte tussen 0 en 12: kleine optielengte tussen de laatste optie-id en de gewenste optie-id
-   13: Voor optielengte van 13 tot 268: Verlengde optielengte is 8bit, dat is de waarde voor optielengte min 13
-   14: voor optielengte van 269 tot 65.804: verlengde optielengte is 16bit, dat is de waarde van de optielengte minus 269
-   15: Gereserveerd voor toekomstig gebruik. Is een fout als het veld Option Delta is ingesteld op 0xFF.

**Optie waarde:**

-   De grootte van het veld Optiewaarde wordt gedefinieerd door de waarde van de optielengte in bytes.
-   Semantisch en formaat dit veld hangt af van de respectievelijke optie.


## Een Aantal Implementaties:
| Name        	| Progamming Language 	| Implemented CoAP version 	| Client/Server                                   	| Implemented CoAP features                                                 	| License 	| Link                                   	|
|-------------	|---------------------	|--------------------------	|-------------------------------------------------	|---------------------------------------------------------------------------	|---------	|----------------------------------------	|
| CoAPthon    	| Python              	| RFC 7252                 	| Client + Server + Forward Proxy + Reverse Proxy 	| Observe, Multicast server discovery, CoRE Link Format parsing, Block-wise 	| MIT     	| https://github.com/Tanganelli/CoAPthon 	|
| Californium 	| Java                	| RFC 7252                 	| Client + Server                                 	| Observe, Blockwise Transfers, DTLS                                        	| EPL+EDL 	| https://www.eclipse.org/californium/   	|
| libcoap     	| C                   	| RFC 7252                 	| Client + Server                                 	| Observe, Blockwise Transfers, DTLS                                        	| BSD/GPL 	| https://github.com/obgm/libcoap        	|

[Hier kan je alle implemenaties vinden](https://en.wikipedia.org/wiki/Constrained_Application_Protocol)

## CoAP-groepscommunicatie 

In veel CoAP-toepassingsdomeinen is het essentieel om de mogelijkheid te hebben om meerdere CoAP-bronnen als een groep aan te spreken, in plaats van elke bron afzonderlijk aan te spreken (bijv. Om alle voor CoAP geschikte lichten in een kamer aan te zetten met een enkel CoAP-verzoek dat wordt geactiveerd door het wisselen van de lichtschakelaar). Om aan deze behoefte te voldoen, heeft de IETF een optionele uitbreiding voor CoAP ontwikkeld in de vorm van een experimentele RFC: Groepscommunicatie voor CoAP - RFC 7390 Deze extensie is afhankelijk van IP-multicast om het CoAP-verzoek aan alle groepsleden te bezorgen. Het gebruik van multicast heeft bepaalde voordelen, zoals het verminderen van het aantal pakketten dat nodig is om het verzoek aan de leden te bezorgen. Multicast heeft echter ook zijn beperkingen, zoals slechte betrouwbaarheid en cachevriendelijk zijn. Een alternatieve methode voor CoAP-groepscommunicatie waarbij gebruik wordt gemaakt van unicasts in plaats van multicasts, is afhankelijk van een tussenpersoon waar de groepen worden gemaakt. Klanten sturen hun groepsaanvragen naar de tussenpersoon, die op zijn beurt individuele unicast-verzoeken naar de groepsleden stuurt, de antwoorden van hen verzamelt en een geaggregeerd antwoord terugstuurt naar de klant.

## Beveiligingsproblemen

Hoewel de protocolstandaard bepalingen bevat om de dreiging van amplificatie-aanvallen te verminderen, worden deze bepalingen in de praktijk niet geïmplementeerd, wat resulteert in de aanwezigheid van meer dan 580.000 doelen die zich voornamelijk in China bevinden en aanvallen tot 320 Gbps 

### **Bronnen**

Constrained Application Protocol. (Juni 2018). In _devopedia_. Geraadpleegd op 30 maart 2020, van  
   [https://devopedia.org/constrained-application-protocol](https://devopedia.org/constrained-application-protocol)
   
Constrained Application Protocol : IOT Part 32. (Oktober, 2018). In _engineersgarage_. Geraadpleegd op 30 maart 2020, van  
   [https://www.engineersgarage.com/tutorials/constrained-application-protocol-iot-part-32/](https://www.engineersgarage.com/tutorials/constrained-application-protocol-iot-part-32/)
   
Constrained Application Protocol. (n.d.). In _Wikipedia_. Geraadpleegd op 30 maart 2020, van  
   [https://en.wikipedia.org/wiki/Constrained_Application_Protocol](https://en.wikipedia.org/wiki/Constrained_Application_Protocol)
   
CoAP. (n.d.). In _coap.technology_. Geraadpleegd op 30 maart 2020, van  
[https://coap.technology/](https://coap.technology/)