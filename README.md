# PAD
PAD-UTM-University-MD / Laborator 2 – 2025
Web Proxy: Realizarea Transparentei în Distribuirea Datelor


  1.Descriere generală:
Acest proiect implementează un Web Proxy inteligent, complet funcțional, care stă între client și multiple servere Data-Warehouse (DW) și oferă:
Load balancing (round-robin)
Caching inteligent pentru GET
Proxy transparent (clientul vede un singur endpoint)
Procesare concurentă (workers / threads)
DW servere cu suport JSON și XML
Interfață unificată pentru acces la date
Proiectul poate fi rulat pe orice sistem Linux/Windows și respectă 100% cerințele laboratorului de la Univer.

  2.Structura proiectului:
  lab2-web-proxy/
│
├── README.md
│
├── src/
│   ├── dw_server.py
│   ├── proxy_server.py
│   └── requirements.txt
│
└── docs/
    ├── arhitectura.png
    ├── flux_proxy.png
    ├── cache_diagram.png
    ├── load_balancer.png
    └── concluzie.png

<img width="492" height="56" alt="Screenshot 2025-11-19 at 14 45 13" src="https://github.com/user-attachments/assets/d573a4ef-0ba7-4177-909f-d989052f9185" />

  
  3.Arhitectura sistemului:

  Sistemul este format din:
Clientul → trimite cereri HTTP (curl, browser, Postman etc.)
Reverse Proxy → distribuie cereri, aplică caching, ascunde infrastructura
Serverele DW (Data Warehouse) → stochează date semi-structurate (employees)
Proxy-ul primește toate cererile și trimite request-urile mai departe către unul din serverele DW folosind round-robin.

  4.Instalare


1. Creează mediu de lucru
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
2. Clonează repo-ul tău GitHub
git clone https://github.com/<user>/lab2-web-proxy.git
cd lab2-web-proxy/src
3. Creează mediul virtual
python3 -m venv venv
source venv/bin/activate
4. Instalează dependențele
pip install -r requirements.txt


Rulare servere DW
DW 1 (port 8001)
export DW_NAME="DW-1"
uvicorn dw_server:app --host 0.0.0.0 --port 8001
DW 2 (port 8002)
export DW_NAME="DW-2"
uvicorn dw_server:app --host 0.0.0.0 --port 8002

Rulare Reverse Proxy
cd src
uvicorn proxy_server:app --host 0.0.0.0 --port 9000 --workers 4

  
  
  5.Testare funcționalități

  1. POST – Creare employee
curl -X POST http://localhost:9000/employees \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "role": "Dev", "salary": 15000}'
Output DW:
[DW-1] POST /employees -> created id=1
2. GET JSON – Lista employees
curl "http://localhost:9000/employees?offset=0&limit=10&format=json"
3. GET XML – Un employee în format XML
curl "http://localhost:9000/employees/1?format=xml"

  

4. Load Balancing demonstrat
Trimite 4 cereri:
curl http://localhost:9000/employees
curl http://localhost:9000/employees
curl http://localhost:9000/employees
curl http://localhost:9000/employees
Log:
[PROXY] Forward -> DW-1
[PROXY] Forward -> DW-2
[PROXY] Forward -> DW-1
[PROXY] Forward -> DW-2

5. Cache demonstrat
Primul GET:
[PROXY] Forward GET -> DW-1
Al doilea GET:
[PROXY] Cache HIT
6. Concurență (thread-per-request)
sudo apt install apache2-utils
ab -n 50 -c 10 http://localhost:9000/employees


!!  Explicații tehnice importante

DDOS protection basic (prin caching)
✔ request mapping către controllere
✔ thread-safe operations
✔ validare format (JSON/XML)
✔ invalidare cache la POST/PUT/DELETE

Unde se folosește acest model în industrie?
Acest laborator reproduce arhitecturi reale folosite în:
1. NGINX Reverse Proxy
Folosite de Google, Netflix, Meta, Amazon.
🟦 2. API Gateway (microservicii)
Kong, AWS API Gateway, Azure APIM funcționează similar:
expun un endpoint
fac routing & caching & balancing
🟦 3. Big Data / Data Warehouse Clusters
Google BigQuery, AWS Redshift, Snowflake folosesc un „proxy” pentru acces la cluster.
🟦 4. Sisteme Enterprise
Orice aplicație cu microservicii → are un gateway exact ca acest  Proxy.

  Concluzie finală
Am implementat o arhitectură Web Proxy complet funcțională, conform cerințelor laboratorului.
Proxy-ul oferă transparență, ascunzând distribuirea datelor între DW-uri.
Load-balancing-ul distribuiă cererile în mod egal.
Caching-ul optimizează accesul și reduce încărcarea backend-urilor.
Serverele DW oferă date în JSON și XML, fiind ușor de integrat.
Modelul reflectă foarte bine arhitecturi enterprise, moderne și robuste.



