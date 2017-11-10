\documentclass{article}
\usepackage[utf8]{inputenc}
\title{En jämförelse av Mark-Sweep mot reference counting för automatisk skräpsamling}
\author{Love Nordling}
\begin{document}
\maketitle
\section{Inledning}
Minnesläckage sker när ett program allokerar minne på den så kallade heapen utan att frigöra det när det inte längre behövs. Programmet tar nämligen i sig inte bort minne som blivit allokerat av sig självt utan detta måste programmeraren själv göra i koden. Om till exempel en pekare till ett allokerat minnesblock försvinner eller skrivs över så kommer detta block av minne att fortsätta ta upp plats. Sker detta i loopar eller i rekursiva delar av ett program så kan minnet snabbt bli fullt. 

Processen att städa bort minnesläckage ur ett program tar ofta upp stora delar av utvecklingstiden då upprensningen av minne inte sker som en direkt följd av att skriva programmet utan programmeraren måste konstant ha minnet i åtanke eller spendera lång tid med att analysera när och var olika pekare tappas bort. För att undvika denna ofta smärtsamma process har metoder för automatisk skräpsamling utvecklats.

\section{Automatisk skräpsamling}
Automatisk skräpsamling avser att ta bort skräp-minne som användaren inte längre använder. Mer konkret ska en skräpsamlare frigöra minnesblock som det inte finns någon kedja av pekare till. 

Det finns många sätt att automatisk rensa skräp från minnet men här kommer två kända och grundläggande Mark Sweep samt Reference Counting att jämföras. 
\section{Mark sweep}
Tanken med mark-sweep är att den inte körs kontinuerligt utan vid enstaka tillfällen för att rensa upp hela minnet. Skräpsamlaren går igenom alla objekt som kan nås från stacken och tar bort de som inte gick att nås. Detta görs i två steg. Ett Mark-stadium där objekt markeras som nåbara eller icke nåbara. Ett Sweep-stadium där de icke nåbara objekten rensas från minnet.
\subsection{Mark-stadium}
Algoritmen börjar med att markera alla objekt i heapen som icke-nåbara. Därefter avgörs vilka objekt som kan nås från stackvariablerna. För varje objekt som ett annat objekt refererar till så markeras det som nåbart samt avgörs vilka detta objekt kan nå.
\subsection{Sweep-stadium}
När inga fler objekt kan markeras som nåbara så kan programmet rensa bort de objekt som inte har fått denna markering. Det görs genom att gå igenom hela heapen och fria allt minne som inte är markerat som nåbart.
\subsection{För- och nackdelar}
En stor fördel med mark-sweep är att minnesläckage förekommer nästan inte alls. Varje objekt eller grupp av länkade objekt som inte har någon referens till sig från stacken kommer att tas bort. Ytterligare en fördel är att prestandaminskningen för programmet ter sig nästintill obefintlig. 

När skräprensningen börjar dock måste programmet stanna upp. Om det är av vikt att programmet är konstant körande och responsivt så kan detta orsaka stora problem för programmets syfte. 


\section{Reference Counting}
Med reference counting vill man undvika att stanna programmet och istället frigöra minne allt eftersom programmet kör. Detta görs genom att, som namnet antyder, räkna antalet gånger som ett varje objekt är refererat och frigöra minnesplatsen om antalet referenser går ner till noll. Denna skräpsamlingsprocess sker konstant vid körning. Så fort en ny pekare eller referens skapas så ökas antalet referenser och så fort en referens/pekare tas bort eller riktas om så minskar antalet referenser för det objekt som utpekades. 

\subsection{För- och nackdelar}
Eftersom att skräpsamlingen sker löpande under programmets körning så sker det aldrig några avbryt. Detta gör att programmeraren med större säkerhet kan göra antaganden och påverka om programmets prestanda.

Dock leder reference counting ofta till att programmet presterar något sämre under körning då den konstanta skräpsamlingen orsakar en del overhead. Är det kritiskt att programmet körs väldigt effektivt så kan detta vara en stor nackdel.

En av de största problemen är dock att reference counting endast kan rensa bort kedjor av objekt som pekar åt ett och samma håll. D.v.s. om två objekt pekar på varandra men är i övrigt helt frånskilda resten av programmet så kommer antalet referenser till dessa objekt aldrig stega ner till noll och därmed skapas här ett minnesläckage. Alltså så fort programmet innehåller en kedja av objekt som går i en cirkel av något slag så kommer inte reference counting att fungera. 

Dock finns det sätt kringgå detta. De mest självklara är att inte tillåta skapandet av loopade strukturer eller att låta programmeraren göra en sorts semimanuell minneshantering där denne endast behöver se till att alla loopade kedjor av objekt bryts upp innan referenserna till dem försvinner. Ett sätt att kunna använda sig av loopade kedjor utan att behöva bry sig om att bryta upp dessa är att använda sig av starka och svaga pekare/referenser. De starka referenserna får endast gå i en riktning och det är dessa som skräpsamlaren tittar på. De svaga referenserna fungerar i övrigt som en vanlig pekare men ignoreras av skräpsamlaren. 

\section{Jämförelse}
De två metoderna kommer med tydliga skillnader i deras fördelar respektive nackdelar. Mark-Sweep är tydligt den som är lättast för programmeraren att använda då denne inte behöver tänka på minnesläckor över huvud taget. Algoritmen tar nämligen hand om i princip alla möjliga minnesläckage. Med referense counting krävs istället att programmeraren antingen designar sitt program efter metodens begränsningar eller använder starka respektive svaga pekare, vilket kan tyckas vara omständigt. 

Gällande prestanda så är metoderna verkligen motpoler till varandra. Mark-Sweep tar väldigt lite av programmets prestanda under körning men måste istället pausa programmet helt när skräpsamling väl sker. Reference countning kräver mer prestanda under körning men görs kontinuerligt och stannar därför aldrig upp programmet. 

Vilken av metoderna som är bäst beror till största del på hur programmet ska fungera men även vad programmeraren har för ambitioner gällande utvecklingstiden. Troligtvis finns det inte någon metod för skräpsamling som är bäst i alla lägen vilket poängterar vikten av att programmerare har en förståelse för hur dessa fungerar så att de kan välja rätt metod för just deras program eller anpassa projektet efter skräpsamlaren. 

\section{Referenser}
\begin{itemize}
\item Mark-and-Sweep: Garbage Collection Algorithm. Chirag Agarwal, 11/10/2017, http://www.geeksforgeeks.org/mark-and-sweep-garbage-collection-algorithm/
\item Back To Basics: Reference Counting Garbage Collection. Abhinaba Basu, 01//27/2009, https://blogs.msdn.microsoft.com/abhinaba/2009/01/27/back-to-basics-reference-counting-garbage-collection/
\end{itemize}
\end{document}
