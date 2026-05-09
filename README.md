Student: Staicu Claudia-Andreea
Grupa: 1147

Link aplicatie: https://proiect-cloud-computing-2026.vercel.app
Link repository: https://github.com/claudiastaicu/Proiect-Cloud_Computing-2026

1. Introducere
   CineSearch este o aplicatie web de tip Single Page Application (SPA) care integreaza trei servicii cloud externe prin intermediul unor API-uri REST. Utilizatorii se pot autentifica, pot cauta filme, pot vizualiza detalii complete (rating, actori, regizor, recenzii) si pot gestiona o lista personala de filme stocata in cloud. Aplicatia este publicata pe platforma Vercel si codul sursa este disponibil pe GitHub.
Scopul aplicatiei este de a agrega informatii din surse multiple intr-o singura interfata moderna si intuitiva, eliminand necesitatea accesarii mai multor platforme separate pentru informatii despre filme.

2. Descrierea problemei
    Utilizatorii interesati de filme trebuie sa acceseze frecvent mai multe platforme (IMDb, Rotten Tomatoes, TMDB) pentru a obtine informatii complete despre un film. Aceasta este o experienta fragmentata si ineficienta.
CineSearch rezolva aceasta problema prin:
•	Agregarea datelor din doua surse cloud (TMDB si OMDb) intr-o singura interfata
•	Posibilitatea de autentificare si persistenta datelor dupa refresh
•	Stocarea unei liste personale de filme in cloud (JSONBin), accesibila de oriunde
•	Interfata rapida, fara reincarcarea paginii, bazata pe requesturi HTTP asincrone

3. Descrierea serviciilor cloud si a API-urilor utilizate
  Aplicatia utilizeaza trei servicii cloud externe, fiecare accesat prin API REST:

3.1 The Movie Database — TMDB API
URL de baza	https://api.themoviedb.org
Tip serviciu	Cloud REST API — baza de date cinematografica
Autentificare	API Key transmis ca parametru in URL (api_key= 8bbd77348419af48a30565ec31a6f20b)
Plan utilizat	Gratuit — acces nelimitat pentru uz educational
Documentatie	https://developer.themoviedb.org/docs

Endpoint-uri utilizate din TMDB:
•	GET /trending/movie/week — filme populare ale saptamanii (afisate la deschiderea aplicatiei)
•	GET /search/movie — cautare film dupa titlu
•	GET /movie/{id} — detalii complete film (gen, durata, buget, incasari, tagline)
•	GET /movie/{id}/credits — echipa filmului (regizor, actori principali)


3.2 OMDb API — Open Movie Database
URL de baza	https://www.omdbapi.com
Tip serviciu	Cloud REST API — date cinematografice suplimentare
Autentificare	API Key transmis ca parametru in URL (apikey= 45df9f6a)
Plan utilizat	Gratuit — 1.000 requesturi/zi
Documentatie	https://www.omdbapi.com

Endpoint-uri utilizate din OMDb:
•	GET /?i={imdb_id} — preia rating IMDb, rating Rotten Tomatoes, plot in engleza, tara de origine

3.3 JSONBin.io — Stocare date in cloud
URL de baza	https://jsonbin.io/app/dashboard
Tip serviciu	Cloud JSON Storage — stocare date structurate in cloud
Autentificare	X-Master-Key transmis in headerul requestului HTTP
Plan utilizat	Gratuit — stocare persistenta in cloud
Documentatie	https://jsonbin.io/api-reference

Endpoint-uri utilizate din JSONBin:
•	POST /v3/b — creaza un bin nou in cloud (la primul login al utilizatorului)
•	GET /v3/b/{id}/latest — citeste lista de filme salvate din cloud
•	PUT /v3/b/{id} — actualizeaza lista (adaugare sau stergere film)

4. Flux de date
4.1 Autentificare si persistenta
Aplicatia implementeaza un sistem de autentificare cu inregistrare si login. Datele utilizatorilor (username si parola) sunt stocate local in browserul utilizatorului (localStorage). La fiecare reincarcarea a paginii, aplicatia verifica daca exista o sesiune activa salvata si relogheaza automat utilizatorul, asigurand persistenta sesiunii dupa refresh.

4.2 Fluxul complet al datelor
1.	Utilizatorul acceseaza aplicatia → se verifica sesiunea salvata in localStorage
2.	Daca nu exista sesiune → se afiseaza ecranul de autentificare
3.	Dupa login → se trimite un request GET catre TMDB pentru filmele trending
4.	La cautarea unui film → GET catre TMDB /search/movie cu titlul introdus
5.	La click pe un film → doua requesturi GET in paralel: detalii film + credite (TMDB), apoi GET catre OMDb pentru rating IMDb si Rotten Tomatoes
6.	La primul login al unui cont nou → POST catre JSONBin pentru a crea un spatiu de stocare in cloud
7.	La adaugarea unui film manual → GET catre JSONBin (citire lista curenta) + PUT (scriere lista actualizata)
8.	La stergerea unui film → GET catre JSONBin (citire lista) + PUT (scriere lista fara filmul sters)

5. Metode HTTP utilizate
Aplicatia utilizeaza urmatoarele metode HTTP pentru comunicarea cu cele trei servicii cloud:

Metodă	Endpoint	Serviciu	Descriere
GET	/trending/movie/week	TMDB	Filme populare ale saptamanii
GET	/search/movie?query=...	TMDB	Cautare film dupa titlu
GET	/movie/{id}	TMDB	Detalii complete film
GET	/movie/{id}/credits	TMDB	Regizor si actori
GET	/?i={imdb_id}	OMDb	Rating IMDb si Rotten Tomatoes
POST	/v3/b	JSONBin	Creare bin nou in cloud (la inregistrare)
GET	/v3/b/{id}/latest	JSONBin	Citire lista filme din cloud
PUT	/v3/b/{id}	JSONBin	Actualizare lista (adaugare/stergere)

5.1 Exemple de request / response
GET — Filme trending (TMDB)
Request:
GET https://api.themoviedb.org/3/trending/movie/week?api_key=KEY&language=ro-RO
Response (simplificat):
{ "results": [{ "id": 1241982, "title": "Moana 2", "vote_average": 7.0, "release_date": "2024-11-27" }] }


GET — Detalii film din OMDb
Request:
GET https://www.omdbapi.com/?i=tt1375666&apikey=KEY
Response (simplificat):
{ "Title": "Inception", "imdbRating": "8.8", "Ratings": [{ "Source": "Rotten Tomatoes", "Value": "87%" }], "Plot": "A thief who steals..." }


POST — Creare bin nou in JSONBin
Request:
POST https://api.jsonbin.io/v3/b
Headers:
Content-Type: application/json
X-Master-Key: [cheia secreta JSONBin]
X-Bin-Name: cinesearch_[username]
Body:
{ "films": [] }
Response:
{ "record": { "films": [] }, "metadata": { "id": "abc123", "createdAt": "...", "name": "cinesearch_claudia" } }
📸 Screenshot: DevTools → Network → filtreaza dupa 'Fetch/XHR' → gaseste requestul POST catre 'api.jsonbin.io' → tab Headers → se vede 'Request Method: POST'

PUT — Adaugare film in lista (JSONBin)
Request:
PUT https://api.jsonbin.io/v3/b/{bin_id}
Body:
{ "films": [{ "title": "Inception", "year": "2010", "genre": "Sci-Fi", "rating": "9", "desc": "...", "addedBy": "UserTest1" }] }
Response:
{ "record": { "films": [...] }, "metadata": { "id": "abc123", "version": 2 } }









5.2 Autentificare si autorizare servicii
•	TMDB: API Key generat din contul de pe themoviedb.org, transmis ca parametru ?api_key= in URL. Nu necesita autentificare OAuth.
•	OMDb: API Key generat din contul de pe omdbapi.com, transmis ca parametru ?apikey= in URL. Plan gratuit: 1.000 requesturi/zi.
•	JSONBin: Secret Key generat din contul de pe jsonbin.io, transmis in headerul HTTP X-Master-Key al fiecarui request. Permite operatii CRUD (POST, GET, PUT) asupra datelor stocate in cloud.

6. Capturi ecran aplicatie (capturile de ecran pot fi vizibile in fisierul pdf incarcat pe platforma)
6.1 Ecran de autentificare


6.2 Inregistrare cont nou

6.3 Pagina principala — filme trending

6.4 Rezultate cautare

6.5 Detalii film — date din TMDB + OMDb

6.6 Adaugare film manual 
6.7 Lista mea — filme salvate in cloud
6.8 Stergere film din lista
6.9 Tab Favorite
7. Publicarea aplicatiei
Aplicatia este publicata pe platforma Vercel, un serviciu cloud de hosting pentru aplicatii web statice si dinamice. Publicarea se realizeaza prin conectarea repository-ului GitHub la Vercel, care genereaza automat un URL public la fiecare commit nou.

•	Platforma de publicare: Vercel (https://vercel.com)
•	URL public: https://proiect-cloud-computing-2026.vercel.app
•	Repository GitHub: https://github.com/claudiastaicu/Proiect-Cloud_Computing-2026
•	Deploy automat: orice modificare push-uita pe GitHub este publicata automat in ~30 secunde

8. Referinte
•	TMDB API Documentation: https://developer.themoviedb.org/docs
•	OMDb API Documentation: https://www.omdbapi.com
•	JSONBin API Documentation: https://jsonbin.io/api-reference
•	Vercel Platform: https://vercel.com
•	GitHub Repository: https://github.com/claudiastaicu/Proiect-Cloud_Computing-2026
