# BussHållplatsTid

Visar de tre nästa bussavgångarna från en hållplats, med data från [ResRobot](https://www.trafiklab.se/api/our-apis/resrobot-v21/) (Trafiklab). Byggd för en liten stående skärm (720×1280) på en Raspberry Pi i kioskläge, men fungerar i vilken webbläsare som helst.

- **Backend:** ASP.NET Core (.NET 10). Hämtar avgångar från ResRobot `departureBoard` och exponerar dem på `GET /api/departures`.
- **Frontend:** Angular 21. Visar nästa tre avgångar och uppdaterar var 30:e sekund.
- **Docker:** en enda image som bygger båda delarna och serverar allt på port 8080.

## Kom igång

Du behöver en API-nyckel för ResRobot v2.1. Den är gratis och skaffas på [trafiklab.se](https://www.trafiklab.se/).

```bash
cp .env.example .env
vim .env          # fyll i ResRobot__ApiKey
```

### Med Docker

```bash
docker build -t busshallplatstid .
docker run --rm -p 8080:8080 --env-file .env busshallplatstid
```

Öppna http://localhost:8080.

### Utan Docker (utveckling)

```bash
# API:t (lyssnar på http://localhost:5196)
cd BussHållplatsTid
dotnet user-secrets set "ResRobot:ApiKey" "<din-nyckel>"
dotnet run

# Frontend i en annan terminal (proxar /api till API:t)
cd frontend
npm ci
npm start
```

Öppna http://localhost:4200.

## Inställningar

| Inställning | Miljövariabel | Standard |
|---|---|---|
| API-nyckel | `ResRobot__ApiKey` | *(måste anges)* |
| Hållplats-id | `ResRobot__StopId` | `740019159` |
| ResRobot-adress | `ResRobot__BaseUrl` | `https://api.resrobot.se/v2.1/` |

Hållplatsens id hittar du med ResRobots `location.name`-anrop. Riktningen som visas, hur många avgångar och hur ofta sidan uppdateras står överst i `frontend/src/app/app.ts` (`direction`, `maxShown`, `refreshMs`).

API:t tar också en valfri `direction`-parameter, till exempel `GET /api/departures?direction=Karskär`.

## Projektstruktur

```
BussHållplatsTid/   ASP.NET Core-API:t (Controllers, Services, Models)
frontend/           Angular-appen
Dockerfile          bygger frontend + API till en image
```

Lägg aldrig `.env` eller en riktig nyckel i `appsettings*.json` i git. `.env` är redan med i `.gitignore` och `.dockerignore`.
