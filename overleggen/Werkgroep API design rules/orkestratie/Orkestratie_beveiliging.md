# Orkestratie beveiliging

 

Aanzet voor een beveiligingsarchitectuur voor orkestratie engines (transparant en niet transparante verwerking)

> Concept v 0.2 – 15-02-2024 – Martin van der Plas

## Context

Orkestratie services bieden over het algemeen een oplossing voor een complexe vraag en een antwoord wat input uit meerdere databronenn bevat.  Conform de [Api Strategie architectuur typologie](https://docs.geostandaarden.nl/api/API-Strategie-architectuur/#systeem-proces-convenience) is het daarmee een zogenaamde "Process API" die meerdere systeem API's aanroept.

Voor de beveiliging van ee ndergelijke process API is het belangrijkste verschil of de Process API kennis heeft van de inhoud en de state van die inhoud ook bijhoudt of dat de Process API geen kennis heeft van de inhoud en daarmee ook geen state hoeft bij te houden. 

Wanneer de Process API geen kennis heeft van de inhoud en geen state vasthoud noemen we dit transparant.

> we focussen in deze context op het bevragen van services en niet op de transactionele kant 
>
> We gaan in onderstaande situaties er vanuit dat er vertrouwelijke gegevens worden bevraagd. Voor het bevragen open data is dergelijke beveiliging niet noodzakelijk.

## Use cases

In deze context onderkennen we vier use cases

-    De Client bevraagt direct de bronnen en heeft de orkestratie daarmee ingebed in de client software (deze use case wordt verder niet uitgediept)
-    De User Identity propageert door tot de bronnen
-    De User Identity wordt onderbroken door de orkestratie
-    De user bevraagt 1 bron en de service van de bron bevraagt nog een andere bron zonder dat de user of client hier weet van heeft (deze use case wordt verder niet uitgediept)



## Transparant - token based

### Rationale:

Er is een direct verband tussen de resources en de identity van de client/gebruiker. Dit is maakt het mogelijk om ook privacy gevoelige data veilig te gebruiken in de orkestratie. 
- In de praktijk wordt deze vorm van orkestratie ook vaak client side gedaan door vanuit de client direct meerdere API's aan te roepen,
- en is er bij de orkestratie service vaak geen sprake van filtering van data,
- het betreft dan meer een bundeling van meerdere bronnen dan echt orkestratie,
- het lijkt bij bevragingen vaak meer op het composition patroon dan een orkestratie patroon.


### Identity:

- Resource A,B&C kennen de Identity van user U 
  (de recources zijn namelijk aan de User gerelateerd of de User is vanuit zijn functie gemachtigd de resources te raadplegen)
- Service Y kent de Identity van user U niet

### Authenticiteit:

- De Authenticiteit van Client X wordt gegarandeerd door Identity Service Provider Z
- De Authenticiteit van Service Y wordt geverifieerd door A,B & C 
- De Authenticiteit van Service Y moet worden geverifieerd door Client X
- De Authenticiteit van Resource A,B & C moet worden geverifieerd door Service Y

### Autorisatie:

- De autorisatie is per Resource dynamisch in te stellen
- Service Y moet overweg kunnen met het feit dat resource A,B & C de autorisatie kunnen weigeren

![Schematische weergave van Transparante-Orkestratie](../../../media/orkestratie-Transparante-Orkestratie.drawio.svg)

### Variatie

1. Identity Service Provider Z kan ook de toegang zijn tot een gefedereerde omgeving van Identity providers waarbij Z door middel van  [OAuth Token Exchange](https://datatracker.ietf.org/doc/html/rfc8693) de access tokens worden ophaald bij de Identity providers van de verschillende resources en deze tokens door de client worden meegegeven bij het request

![Schematische weergave van Federatieve Transparante-Orkestratie](../../../media/orkestratie-Federatief-Transparante Orkestratie.drawio.svg)

### Implicaties transparante orkestratie:

- Er is een sterke vertrouwensband nodig tussen zowel de Client & Service alsook de Service en de Resources. Deze vertrouwensband zal waarschijnlijk worden vertaald naar eisen, audits en contracten om te voorkomen dat de Service Y toch de gegevens inziet die de resouces terugsturen aan de client.

- Organisatorisch zullen X, Y, Z, A & B praktisch gezien niet allemaal van verschillende organisaties zijn. Het is logischer wanneer A, B & Y van 1 organisatie zijn of wanneer X & Y van 1 organisatie zijn. Service Y wordt namelijk alleen ontwikkeld als er een vraag is en een voordeel voor de resources of de client.

- Afhankelijk van de situatie is het waarschijnlijk dat Identity provider Z een algemene dienst is van de overheid (denk aan DigiD of eHerkenning) of dat dit een dienst is van 1 van de organisaties in de orkestratie.

 

## Niet Transparant - token based

### Rationale:

- er is een ontkoppeling tussen de resources en de client. Dit is geen probleem bij open data maar wel bij Privacy gevoelige data. 
- In de praktijk is dit wel hoe bijvoorbeeld een huisarts werkt en hoe veel balies en overheidsorganisaties werken.
- De token / OAuth flow die wordt geschetst kan in deze situaties ook een andere vorm van authenticatie of identificatie zijn 
- Deze situatie is vaak in gebruik binnen 1 organisatie waarbij alle onderdelen van de orkestratie onder verantwoording van 1 organisatie vallen en de User ook in dienst is bij deze organisatie

 

### Identity:

- Resource A,B & C kennen de identity van Service Y, niet van Client X of User U

- Service Y kent de Identity van User U en of Client X

 

### Authenticiteit:

- De Authenticiteit van Client X wordt gegarandeerd door Identity Service Provider Z

- De Authenticiteit van Client X wordt geverifieerd door Service Y en Identity Service Provider Z

- De Authenticiteit van Service Y moet worden geverifieerd door Client X

- De Authenticiteit van Resource A,B & C moet worden geverifieerd door Service Y

 

### Autorisatie:

- De autorisatie is bepaald in en door de Service provider Y, Resource A,B & C weten niet dat hun data wordt gedeeld met Client X en User U of een andere client of user

- Service Y moet geautoriseerd zijn bij Resource A,B & C om namens Client X en/of User U data op te vragen

- Resource A,B & C delegeren toegang tot hun data aan Service Y 

![Schematische weergave van Niet Transparante-Orkestratie](../../../media/orkestratie-Niet-transparante-Orkestratie.drawio.svg)

### Implicaties:

- Er is een sterke vertrouwensband nodig tussen zowel de Client & Service alsook de Service en de Resources. Deze vertrouwensband zal waarschijnlijk worden vertaald naar eisen, audits en contracten.
- De aanbieder van Service Y, stel een gemeente, zal zorgvuldig moeten borgen en vastleggen dat de service die ze aanbieden alleen gegevens opvraagt bij de resources wanneer daar ook een expliciete vraag voor is van de User U. Er moet worden voorkomen dat in deze situatie een kwaadwillende toegang krijgt tot de service en daarmee alle resources van alle users kan bevragen.
- Deze oplossing is technisch waarschijnlijk makkelijker te realiseren en vaker in gebruik maar kent ook veel grotere risico's dan de transparante orkestratie.

 