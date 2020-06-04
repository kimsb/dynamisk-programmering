# Dynamisk programmering

Her skal vi øve oss på litt dynamisk programmering ved å jobbe med Fibonacci-følgen og knapsack-problemet!

Enkelt forklart så er dynamisk programmering en generell metode som hjelper oss med å løse større problemer ved å dele dem opp i mindre problemer,
løse disse mindre problemene hver for seg, og deretter bruke resultatene fra dette i løsningen av det større problemet.

Dere kan lese mer om dynamisk programmering og hva slags problemer som lar seg løse ved hjelp av dynamisk programmering på [wikipedia](https://en.wikipedia.org/wiki/Dynamic_programming)

En vanlig fremgangsmåte er å
- finne en naiv, rekursiv algoritme som løser et mindre testsett
- legge til [memoisering](https://en.wikipedia.org/wiki/Memoization) (lagring av resultater på delproblemer underveis) for øke ytelse
- lage en "bottom-up"-implementasjon for å kvitte seg med rekursive kall

OK, la oss teste det ut!

# [Fibonacci-følgen](https://no.wikipedia.org/wiki/Fibonaccitall)
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...]

Oppgaven går ut på å lage en metode `fib(n)` som tar et tall 'n' og returnerer tallet i fibonacci-følgen som ligger på indeks 'n'.

- eks: fib(0) -> 0
- eks: fib(5) -> 5
- eks: fib(10) -> 55

"Bortsett fra de to første startverdiene 0 og 1 framkommer leddene i følgen ved å summere de to forrige leddene"

Det vil si at vi kan finne fib(n) ved å legge sammen fib(n-1) + fib(n-2)

Hvis dere vil kan dere se på denne [youtube-filmen](https://www.youtube.com/watch?v=vYquumk4nWw) som forklarer konseptet ganske så fint.

## Vi starter med å lage den naive, rekursive fremgangsmåten:
- hvis n == 0 eller 1, returner n,
- ellers: returner fib(n-1) + fib(n-2)

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
fun fib(n: Int): BigInteger {
    if (n < 2) {
        return n.toBigInteger()
    }
    return fib(n - 1) + fib(n - 2)
}
  ```
  
</details>

Dette funker jo fint!
...men kun for veldig små 'n'. Allerede ved n = 40 begynner dette å gå alt for tregt...
Denne fremgangsmåten har en kompleksitet på O(2^n), det vil si at tidsbruken øker eksponensielt.

## La oss speede opp ved å legge til memoisering

- lag et array med størrelse n+1 og bruk det til å lagre alle de midlertidige resultatene.
- hvis array[n] allerede har blitt regnet ut, returner resultatet direkte
- ellers - gjør som vi gjorde i den naive, rekursive metoden

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
val array: Array<BigInteger?> = arrayOfNulls(n + 1)
fun fib(n: Int): BigInteger {
    if (array[n] != null) {
        return array[n]!!
    }
    if (n < 2) {
        return n.toBigInteger()
    }
    array[n] = fib(n - 1) + fib(n - 2)
    return array[n]
}
  ```
  
</details>

Denne algoritmen funker mye bedre, og har en kompleksitet på O(n), altså lineær tid.
Det går med andre ord ganske kjapt, og fib(1 000) kjøres med null problemer! ...men, med stor 'n' blir det veldig mange rekursive kall som legges på call-stacken.
I mitt eksempel, som er implementert med Kotlin, så kræsjer koden om jeg f.eks prøver meg på fib(10 000)

## Hva om vi heller prøver bottom-up?
Da kan vi droppe de rekursive kallene og heller starte ved n = 0 og fylle arrayet opp til og med n.

- hvis n < 2, returner n
- ellers: lag et array med størrelse n+1 og fyll array[0] = 0 og array[1] = 1
- loop fra 2 til n og fyll opp arrayet
- returner array[n]

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
fun fib(n: Int): BigInteger {
    if (n < 2) {
        return n.toBigInteger()
    }
    val array = arrayOfNulls<BigInteger>(n + 1)
    array[0] = BigInteger.ZERO
    array[1] = BigInteger.ONE
    for (i in 2..n) {
        array[i] = array[i - 1]!! + array[i - 2]!!
    }
    return array[n]!!
}
  ```
  
</details>

Nå begynner vi å snakke! fib(10 000) fungerer uten problemer, det gjør fib(100 000) også!
...men det blir jo et himla stort array etterhvert... I mitt eksempel, som er implementert i Kotlin, så kræsjer koden med OutOfMemoryError om jeg prøver meg på fib(1 000 000)

## Kan vi optimalisere enda mer?

Når vi går bottom-up i tilfellet med Fibonacci-følgen, så bruker vi jo bare de to forrige verdiene fra arrayet (for å finne fib(n) trenger vi kun fib(n-1) og fib(n-2)).
Det betyr at vi egentlig kan skrote hele arrayet og bare spare på de to forrige verdiene vi har regnet oss frem til!

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
fun fib(n: Int): BigInteger {
    if (n < 2) {
        return n.toBigInteger()
    }
    var iMinusOne = BigInteger.ONE
    var iMinusTwo = BigInteger.ZERO
    var iCurrent = BigInteger.ZERO
    for (i in 2..n) {
        iCurrent = iMinusOne + iMinusTwo
        iMinusTwo = iMinusOne
        iMinusOne = iCurrent
    }
    return iCurrent
}
  ```
  
</details>

Kult!
Nå funker faktisk fib(1 000 000), selv om det tar noen sekunder...
Shit, det resultatet er et stoooort tall!

# Knapsack 0-1 

Her kan du lese mer om [knapsack-problemet](https://en.wikipedia.org/wiki/Knapsack_problem)
Knapsack 0-1 er en enkel versjon av problemet, hvor et element enten blir ignorert (0) eller valgt (1)

Vi har n antall elementer og 'sekken' vår har en kapasitet c (maks vekt).
De n elementene har en vekt og en verdi, som ligger i to korresponderende arrays av størrelse n+1

Oppgaven er altså å plukke ut de elementene som har plass i sekken, og som gir høyest samlet verdi.

Her er en [youtube-film](https://www.youtube.com/watch?v=xOlhR_2QCXY&t) som forklarer konseptet ganske så fint:

## Først den naive rekursive algoritmen
Vi starter med en peker bakerst på arrayet, og så går vi gjennom alle elementene og gjør begge valgene:
- IKKE ta med elementet - (pekeren flyttes til n-1)
- ta med elementet - (reduser kapasitet og øk samlet verdi før pekeren flyttes til n-1)

Når vi da har gått gjennom hele arrayet har vi sjekket alle mulige kombinasjoner, og får høyest samlet verdi returnert.

Testdata:
vekter = [0,1,2,4,2,5],
verdier = [0,5,3,5,3,2],
n = 5,
c = 10

Dette skal gi en samlet sum på 16

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
fun knapsack(n: Int, c: Int): Int {
    if (n == 0 || c == 0) {
        return 0
    }
    if (vekter[n] > c) {
        return knapsack(n - 1, c)
    }
    val temp1 = knapsack(n - 1, c)
    val temp2 = verdier[n] + knapsack(n - 1, c - vekter[n])
    return max(temp1, temp2)
}
  ```
  
</details>

Dette funker jo utmerket! ...men som tidligere vil denne algoritmen gi oss en kompleksitet på O(2^n) og vi får problemer med større testsett...

## La oss legge til memoisering!

Vi har maksimalt n * c mulige kombinasjoner av elementer,
så la oss lage et todimensjonalt array[n+1][c+1] hvor vi kan lagre resultater underveis.
I Koden vår kan vi først sjekke om vårt array inneholder et resultat, og returnerer i så fall det.
Hvis ikke gjør vi som tidligere, og lagrer resultatene underveis i arrayet

<details>
  <summary>Eksempel på implementasjon i Kotlin</summary>
    
  ```kotlin
val matrix: Array<IntArray> = Array(n + 1) { IntArray(n + 1) }
fun knapsack(n: Int, c: Int): Int {
    if (matrix[n][c] != 0) {
        return matrix[n][c]
    }
    if (n == 0 || c == 0) {
        return 0
    }
    if (vekter[n] > c) {
        return knapsack(n - 1, c)
    }
    val temp1 = knapsack(n - 1, c)
    val temp2 = verdier[n] + knapsack(n - 1, c - vekter[n])

    matrix[n][c] = max(temp1, temp2)
    return matrix[n][c]
}
  ```
  
</details>

Dette gir oss en kompleksitet på O(n) som er mye bedre!
...men her også vil vi få trøbbel med mange rekursive kall ved større testsett...

## Ønsker du noen tøffere utfordringer?
Finn en bottom-up implementasjon av den algoritmen vi har laget nå, og prøv deg på testsettet med 10000 elementer.
Med en kapasitet på 49877 skal høyeste samlede verdi bli 563647

Eller prøv deg på mer komplekse varianter av knapsack-problemet! 💪
