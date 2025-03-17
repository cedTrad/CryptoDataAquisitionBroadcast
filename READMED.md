# Service de Flux de Données Binance

Ce projet fournit un service de collecte, stockage et distribution de données de marché Binance via une interface Redis.

## Architecture

Le projet est composé de plusieurs composants :

1. **BinanceConnector** : Se connecte à l'API WebSocket de Binance pour récupérer les données en temps réel
2. **RedisStorage** : Stocke les données dans Redis et gère les abonnements
3. **FluxManager** : Coordonne les connecteurs et le stockage
4. **FluxService** : Service principal qui expose une interface de commande via Redis
5. **FluxClient** : Client pour interagir avec le service

L'architecture utilise Redis comme bus de communication bidirectionnel :
- Les commandes sont envoyées sur des canaux Redis dédiés
- Les réponses sont publiées sur des canaux spécifiques à chaque requête
- Le statut du service et des flux est stocké dans des hashes Redis

## Installation

### Prérequis

- Python 3.8+
- Redis
- Docker et Docker Compose (optionnel)

### Installation manuelle

1. Cloner le dépôt :
   ```
   git clone https://github.com/votre-utilisateur/binance-flux-service.git
   cd binance-flux-service
   ```

2. Installer les dépendances :
   ```
   pip install -r requirements.txt
   ```

### Installation avec Docker

1. Cloner le dépôt :
   ```
   git clone https://github.com/votre-utilisateur/binance-flux-service.git
   cd binance-flux-service
   ```

2. Lancer les services avec Docker Compose :
   ```
   docker-compose up -d
   ```

## Utilisation

### Démarrer le service

```
python flux_service.py
```

Options disponibles :
- `--batch-size` : Taille des lots de données (par défaut : 1500)

### Utiliser le client en ligne de commande

#### Lister les flux actifs
```
python flux_client.py list
```

#### Créer un nouveau flux
```
python flux_client.py create btcusdt,ethusdt 1m
```

#### Arrêter un flux
```
python flux_client.py stop 1m_btcusdt-ethusdt
```

#### Obtenir le statut d'un flux
```
python flux_client.py status 1m_btcusdt-ethusdt
```

#### Récupérer des données
```
python flux_client.py data btcusdt 1m 100
```

#### Surveiller la santé du service
```
python flux_client.py monitor
```

#### Observer les mises à jour en temps réel
```
python flux_client.py watch btcusdt 1m
```

### Utiliser la bibliothèque client

```python
from flux_client import FluxClient

# Créer un client
client = FluxClient(redis_host="localhost", redis_port=6379)

# Créer un flux
response = client.create_flux(symbols=["btcusdt", "ethusdt"], interval="1m")
flux_id = response["flux_id"]

# Récupérer des données
data = client.get_data("btcusdt", "1m", limit=100)

# S'abonner aux mises à jour
def on_update(data):
    print(f"Mise à jour pour {data['symbol']}: {data['rows']} bougies")

pubsub = client.subscribe_to_updates("btcusdt", "1m", on_update)

# Fermer le client
client.close()
```

## Structure du projet

```
.
├── dataEngine/
│   ├── __init__.py
│   ├── connector.py       # BinanceConnector
│   ├── storage.py         # RedisStorage
│   └── manager.py         # FluxManager
├── flux_service.py        # Service principal
├── flux_client.py         # Client d'interaction
├── requirements.txt       # Dépendances
├── docker-compose.yml     # Configuration Docker Compose
├── Dockerfile.service     # Dockerfile pour le service
└── README.md              # Documentation
```

## Monitoring et santé

Le service publie régulièrement son statut de santé dans Redis :

- Statut global : clé `status:service`
- Statut par flux : clé `status:flux:{flux_id}`

Ces informations peuvent être récupérées directement avec le client Redis ou via le client du service.

## Architecture des commandes Redis

Les commandes sont envoyées sur des canaux Redis spécifiques :

- `commands:flux:create` - Créer un flux
- `commands:flux:stop` - Arrêter un flux
- `commands:flux:list` - Lister les flux
- `commands:data:get` - Récupérer des données
- `commands:status:get` - Récupérer le statut

Les réponses sont publiées sur :
- `responses:{action}:{request_id}`

## Licence

Ce projet est sous licence MIT.