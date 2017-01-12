# 1dv023 - Komplettering för examination 3
##Webhooks
Timothy Fitz kallar webhooks för [”user-defined HTTP callbacks”](http://timothyfitz.com/2009/02/09/what-webhooks-are-and-why-you-should-care/). Det är en server-till-server-teknik för att underlätta integration och möjliggöra att en server kan reagera på en händelse på en annan server.

En webhook är kopplad till en händelse. Man ger en payload URL till ett webhook API och anger vilken/vilka händelser man är intresserad av och så kommer det att skickas en HTTP POST till den angivna URL:en när händelsen inträffar.

Webhooken ger alltså dess användare möjlighet att uttrycka att en notifiering ska ske till ett, via URL:en, användarspecificerat ställe, vid en viss händelse. Exempelvis så använder Github webhooks för att kunna erbjuda [”prenumeration” på händelser](https://developer.github.com/webhooks/), medan Slack beskriver sitt API som [”a simple way to post messages from external sources into Slack”](https://api.slack.com/incoming-webhooks).

Jag provade att använda Githubs webhooks API och fick [det här svaret](https://gist.github.com/sehnpaa/24fab661d9049d7619dffd4ab0f72aa0) när jag provade att skapa en ”new milestone” på mitt sehnpaa/gentodo-repository. Med http://95.85.13.92:8080 som payload URL kunde jag ta emot en POST efter att ha startat [Netcat](https://en.wikipedia.org/wiki/Netcat) och lyssnat med ”nc -l -p 8080”.

Tyvärr verkar inte webhooks ha fått det genomslag som man hoppats på, bl.a. så uttrycker [Timothy själv och andra](http://timothyfitz.com/2009/02/09/what-webhooks-are-and-why-you-should-care/) viss besvikelse över nuläget i kommentarerna.

##Hur vet jag att koden i beroenden är säker?
Det finns sidor, t.ex. snyk.io, som listar upptäckta säkerhetshål i npm-paket. Man kan, med den tjänsten, följa vilka paket som listas och därefter se till att de blir uppgraderade. Snyk finns också installerbart med npm och kan integreras med projektets byggprocess.

RisingStack har skrivit ett omfattande [blogginlägg](https://snyk.io/vuln/) där de listar åtgärder man kan ta till för att öka säkerheten: a) se till att rensa bort paket man inte använder, b) kolla att beroendena används av andra; ju fler som använder paketet desto större är chansen att brister hittas och rättas, c) kolla att man har rätt versioner av beroendena; ofta hjälper det avsevärt att se till att man har senaste versionen, d)  kolla om paketet är övergivet; verkar det vara någon som håller det uppdaterat? och e) välja paket som har fler maintainers minskar risken för att paketet överges. De listar också en del npm-kommando som kan underlätta: ”npm ls” kan användas för lista beroenden, ”npm outdated” för att visa utdaterade paket, ”npm view <package> time.modified” för att visa när paketet senast modifierades och ”npm view <pkg> maintainers” för att lista vilka (och hur många) maintainers paketet har.

Kulturen i Node.js-ekosystemet har uppmuntrat till många, små paket, vilket, tillsammans med en explosion i populäritet, lett till så många paket hamnat i det dolda samtidigt som de används. [RisingStack hävdade](https://blog.risingstack.com/controlling-node-js-security-risk-npm-dependencies/) (för fem månader sedan) att 15% av alla npm-paket har säkerhetshål.

Vill man få kontroll över vilka versioner som kommer in när man uppdaterar med npm så kan man specificera exakta versionsnummer i packages.json. Då kan man uppgradera till endast versioner som man litar på. Uppskattningsvis kostar det nog mer än det smakar, och den alternativ syn på saken, att följa de senaste versionerna av beroendena genom att ständigt hålla sig uppdaterad, är gissningsvis en bättre tradeoff generellt.

##Websocket vs XHR long polling
Dessa tekniker kan används för att lösa problemet med att en sida av kommunikationen inte får reda på när den andra sidan har ny information utan att hela tiden behöva fråga (vilket är ineffektivt).

###Websockets
Använder HTTP för handskakning, vilket är bra för bakåtkompatibilitet, för kommunikationen kan sedan ske via websocket (ws). Endast en TCP-connection behövs och data kan sedan skickas åt båda hållen och både klienten och servern kan själva välja när.

Trafiken går över port 80 vilket underlättar eftersom risken minskar att eventuella, mellanliggande brandväggar behöver konfigureras.

###XHR Long polling
I long polling så börjar dock klienten med att skicka en HTTP request till servern. Istället för att servern svarar direkt så väntar den tills den antingen har ny information eller tills timeout är nodd, och skickar då en HTTP response tillbaka till klienten. Klienten kan välja mellan att nöja sig där eller lyssna vidare genom att återstarta processen.

Om klienten hade bara frågat, fått svar att det inte finns något nytt, frågat igen och fått samma svar osv tills den får ny information så hade det krävs onödigt mycket kommunikation. Lösningen med Long polling är att servern väntar med att svara. På så sätt har ny data chans att dyka upp på servern  medan den väntar. Den nya datan kan då skickas direkt till klienten.

###Jämförelse
XHR Long polling är lite mer ”hack-lösningen”, men har används länge av många och är väl beprövat, medan websockets är den modernare och smidigare lösningen där man inte behöver simulera att tvåvägskommunikation.

Med ws-protokollet [minskas både latency och overhead](http://websocket.org/quantum.html) och websocket.org beskriver själva hur minskningen i overhead ger enorma bandbreddbesparingar när massor av klienter efterfrågar ny data ofta. Med XHR Long polling är man kvar med HTTPs stora request- och response-paket.

Med XHR Long polling har man möjlighet att köra över HTTPS för kryptering, medan man med websocket har sitt eget, liknande protokoll/lager, vid namn wss.
