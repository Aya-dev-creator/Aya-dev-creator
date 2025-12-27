# 🏗️ Diagrammes d'Architecture - Système IA de Surveillance de la Qualité de l'Air

## 1. Architecture Système Générale

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SYSTÈME DE SURVEILLANCE DE L'AIR                  │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   CAPTEURS      │    │   RASPBERRY PI  │    │     CLOUD       │         │
│  │                 │    │                 │    │                 │         │
│  │  • MQ-135       │    │  • Collecte     │    │  • Stockage     │         │
│  │  • DHT11        │    │  • Traitement   │    │  • Analyse      │         │
│  │  • GPS NEO-6M   │    │  • ML Local     │    │  • ML Cloud     │         │
│  │                 │    └─────────┬───────┘    └─────────┬───────┘         │
│  └─────────────────┘             │                      │                  │
│                                  │                      │                  │
│  ┌─────────────────┐             │                      │                  │
│  │   DONNÉES       │◄────────────┼──────────────────────┼─────────────────┐
│  │   TEMPS RÉEL    │             │                      │                 │
│  └─────────────────┘             │                      │                 │
│                                  │                      │                 │
│  ┌─────────────────┐    ┌────────▼───────┐    ┌────────▼───────┐         │
│  │   DASHBOARD     │    │   API BACKEND  │    │   ALERTES      │         │
│  │   WEB/MOBILE    │    │   FLASK/FASTAPI│    │   PUSH/SMS     │         │
│  └─────────────────┘    └────────────────┘    └────────────────┘         │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   PRÉDICTIONS   │    │   ANALYTIQUES   │    │   RAPPORTS      │         │
│  │   ML MODELS     │    │   TEMPS RÉEL    │    │   HISTORIQUES   │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2. Architecture Matérielle - Connexions GPIO Raspberry Pi 4

```
RASPBERRY PI 4 - PINOUT GPIO
═══════════════════════════════════════════════════════════════════════════════

                    3.3V  [1] [2]  5V
        SDA1 (I2C)  [3] [4]  5V
        SCL1 (I2C)  [5] [6]  GND
 GPIO 4 (DHT11)     [7] [8]  GPIO 14 (TX GPS)
                   GND [9] [10] GPIO 15 (RX GPS)
GPIO 17 (MQ135 DO) [11][12] GPIO 18 (PWM)
GPIO 27            [13][14] GND
GPIO 22            [15][16] GPIO 23
                   3.3V[17][18] GPIO 24
        MOSI (SPI) [19][20] GND
        MISO (SPI) [21][22] GPIO 25
        SCLK (SPI) [23][24] CE0 (SPI)
                   GND [25][26] CE1 (SPI)
        SDA0 (I2C) [27][28] SCL0 (I2C)
                   GND [29][30] GPIO 5
GPIO 6             [31][32] GPIO 12
GPIO 13            [33][34] GND
GPIO 19            [35][36] GPIO 16
GPIO 26            [37][38] GPIO 20
                   GND [39][40] GPIO 21

CONNEXIONS CAPTEURS:
─────────────────────

MQ-135 (Qualité Air):
├── VCC  → 5V (Pin 2 ou 4)
├── GND  → GND (Pin 6, 9, 14, 20, 25, 30, 34, 39)
├── AOUT → MCP3008 ADC (SPI)
└── DOUT → GPIO 17 (Pin 11)

DHT11 (Température/Humidité):
├── VCC  → 3.3V (Pin 1 ou 17)
├── GND  → GND
└── DATA → GPIO 4 (Pin 7)

GPS NEO-6M:
├── VCC  → 5V
├── GND  → GND
├── TX   → GPIO 15 (RX) (Pin 10)
└── RX   → GPIO 14 (TX) (Pin 8)

ADS1115 ADC (pour MQ-135 analogique):
├── VCC  → 3.3V
├── GND  → GND
├── SCL  → SCL (Pin 5)
├── SDA  → SDA (Pin 3)
├── A0   → MQ-135 AOUT
└── ADDR → GND (adresse 0x48)
```

## 3. Architecture Logicielle

```
ARCHITECTURE EN COUCHES
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│                           COUCHE PRÉSENTATION                              │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   DASHBOARD     │    │   API REST      │    │   MOBILE APP    │         │
│  │   HTML/CSS/JS   │    │   FLASK/FASTAPI │    │   REACT NATIVE  │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────────────────────────────────────────────┐
│                           COUCHE APPLICATION                               │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   GESTION ML    │    │   TRAITEMENT    │    │   ALERTES        │         │
│  │   SCIKIT-LEARN  │    │   DONNÉES       │    │   TWILIO/SMTP    │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────────────────────────────────────────────┐
│                           COUCHE DOMAINE                                   │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   MODÈLES       │    │   RÈGLES        │    │   VALIDATION     │         │
│  │   ENTITÉS       │    │   MÉTIER        │    │   DONNÉES        │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
┌─────────────────────────────────────────────────────────────────────────────┐
│                           COUCHE INFRASTRUCTURE                            │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   BASE DE       │    │   CLOUD         │    │   CAPTEURS       │         │
│  │   DONNÉES       │    │   SERVICES      │    │   GPIO/I2C/SPI   │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 4. Flux de Données

```
FLUX DE DONNÉES TEMPS RÉEL
═══════════════════════════════════════════════════════════════════════════════

CAPTEURS → RASPBERRY PI → TRAITEMENT → STOCKAGE → ANALYSE → PRÉDICTION → ALERTES
    │           │             │           │         │           │         │
    ▼           ▼             ▼           ▼         ▼           ▼         ▼

• MQ-135    • Lecture     • Filtrage   • SQLite   • ML        • IA       • Push
• DHT11     • GPIO        • Validation • Local    • Stats     • Modèle   • SMS
• GPS       • I2C/SPI     • Normal.    • Cloud    • Patterns  • Prédic.  • Email
• Environ.  • Serial      • Agrégation • Firebase • Tendances • Alertes  • Dashboard

PROCESSUS DE TRAITEMENT:
────────────────────────

1. ACQUISITION
   ├── Lecture capteurs (1Hz - 10Hz)
   ├── Validation données brutes
   └── Timestamp + géolocalisation

2. PRÉTRAITEMENT
   ├── Filtrage bruit
   ├── Calibration capteurs
   ├── Normalisation valeurs

3. STOCKAGE
   ├── Base locale (SQLite)
   ├── Synchronisation cloud
   └── Historique rolling (7 jours)

4. ANALYSE
   ├── Statistiques temps réel
   ├── Détection anomalies
   ├── Calcul AQI (Air Quality Index)

5. PRÉDICTION
   ├── Modèle ML entraîné
   ├── Prévision 24h/7j
   ├── Seuils d'alerte dynamiques

6. NOTIFICATION
   ├── Alertes push mobile
   ├── SMS/email critiques
   └── Dashboard temps réel
```

## 5. Architecture Cloud et IoT

```
ARCHITECTURE CLOUD DISTRIBUÉE
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│                           PLATEFORME CLOUD                                 │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   FIREBASE      │    │   AWS IOT       │    │   AZURE IOT     │         │
│  │   REALTIME DB   │    │   CORE          │    │   HUB           │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
│           │                 │                      │                       │
│           └─────────────────┼──────────────────────┘                       │
│                             │                                              │
│  ┌─────────────────┐        │         ┌─────────────────┐                  │
│  │   MQTT BROKER   │◄───────┼─────────┤   API GATEWAY   │                  │
│  │   ECLIPSE       │        │         │   REST/GRAPHQL  │                  │
│  │   MOSQUITTO     │        │         └─────────────────┘                  │
│  └─────────────────┘        │                                              │
│                             │                                              │
│  ┌─────────────────┐    ┌───▼─────┐    ┌─────────────────┐                  │
│  │   EDGE COMPUTING│    │  ML     │    │   BIG DATA      │                  │
│  │   RASPBERRY PI  │    │  MODELS │    │   ANALYTICS     │                  │
│  └─────────────────┘    └─────────┘    └─────────────────┘                  │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   MONITORING    │    │   LOGGING       │    │   SECURITY      │         │
│  │   DASHBOARDS    │    │   ELASTICSEARCH │    │   AUTHENTICATION │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘

RÉSEAU IoT:
───────────

• PROTOCOLES: MQTT, HTTP/HTTPS, WebSocket
• SÉCURITÉ: TLS 1.3, certificats X.509, JWT
• CONNECTIVITÉ: WiFi, Ethernet, 4G/5G (optionnel)
• LATENCE: < 100ms pour données critiques
• BANDE PASSANTE: Optimisée (compression données)

ARCHITECTURE MICROSERVICES:
───────────────────────────

┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  COLLECTOR  │    │  PROCESSOR  │    │  ANALYZER  │
│  SERVICE    │    │  SERVICE    │    │  SERVICE   │
│             │    │             │    │            │
│ • Capteurs  │    │ • Validation │    │ • ML       │
│ • GPIO      │    │ • Filtrage  │    │ • Prédic.  │
│ • MQTT Pub  │    │ • Agrégation│    │ • Alertes  │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                ┌─────────────┐
                │  API GATEWAY│
                │  SERVICE    │
                └─────────────┘
```

## 6. Maquettes Interface Utilisateur

### Dashboard Principal

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                           AIR QUALITY MONITORING                           ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                          METRICS TEMPS RÉEL                            │ ║
║  │                                                                         │ ║
║  │  AQI: 85 (MODÉRÉ)     TEMP: 22°C     HUM: 65%     PM2.5: 45 µg/m³     │ ║
║  │  [████████░░░░]       [███████░░░]   [████████░░]  [███████░░░]        │ ║
║  │                                                                         │ ║
║  │  LOCALISATION: Paris, France    DERNIÈRE MAJ: 14:32:15                 │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  ┌─────────────────────────────────┬───────────────────────────────────────┐ ║
║  │         GRAPHIQUE AQI           │        CARTE GÉOGRAPHIQUE             │ ║
║  │                                 │                                       │ ║
║  │          ▲                      │  ┌─────────────────────────────────┐  │ ║
║  │        100                      │  │         ZONE ACTUELLE           │  │ ║
║  │                                 │  │                                 │  │ ║
║  │         50 ────────────────────── │  │          ● (Vous êtes ici)     │  │ ║
║  │                                 │  │                                 │  │ ║
║  │          0                      │  │  ZONES À RISQUE:                │  │ ║
║  │        00:00  06:00  12:00  18:00│  │  🔴 Critique                   │  │ ║
║  │                                 │  │  🟠 Élevé                       │  │ ║
║  │                                 │  │  🟡 Modéré                      │  │ ║
║  │                                 │  │  🟢 Bon                         │  │ ║
║  └─────────────────────────────────┴───────────────────────────────────────┘ ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                          PRÉDICTIONS 24H                               │ ║
║  │                                                                         │ ║
║  │  HEURE │ AQI │ QUALITÉ │ TEMP │ HUM │ VENT │ PRÉCIP │ RECOMMANDATION   │ ║
║  │ ───────┼─────┼─────────┼──────┼─────┼──────┼────────┼───────────────── │ ║
║  │  15:00 │ 78  │ MODÉRÉ  │ 23°C │ 62% │ 5km/h│ 0%     │ Masque optionnel │ ║
║  │  16:00 │ 82  │ MODÉRÉ  │ 24°C │ 58% │ 7km/h│ 0%     │ Activité normale │ ║
║  │  17:00 │ 95  │ MAUVAIS │ 25°C │ 55% │ 8km/h│ 5%     │ Réduire activité │ ║
║  │  18:00 │ 110 │ MAUVAIS │ 26°C │ 52% │ 6km/h│ 10%    │ Éviter extérieur │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  [ALERTES] [HISTORIQUE] [PARAMÈTRES] [À PROPOS]                            ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### Page d'Alertes

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                              SYSTÈME D'ALERTES                            ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                          ALERTES ACTIVES                               │ ║
║  │                                                                         │ ║
║  │  🚨 QUALITÉ AIR CRITIQUE - AQI: 150 (TRÈS MAUVAIS)                     │ ║
║  │     Recommandation: Restez à l'intérieur, portez un masque N95         │ ║
║  │     Zone affectée: Paris Centre                                        │ ║
║  │     Durée estimée: 4-6 heures                                          │ ║
║  │                                                                         │ ║
║  │  ⚠️  PIC DE POLLUTION PRÉVU - Dans 2 heures                             │ ║
║  │     AQI prévu: 120-140                                                 │ ║
║  │     Cause: Trafic intense + conditions météo                           │ ║
║  │                                                                         │ ║
║  │  ℹ️  AMÉLIORATION ATTENDUE - Qualité en amélioration                    │ ║
║  │     AQI actuel: 85 → prévu: 65 dans 3h                                 │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                         CONFIGURATION ALERTES                          │ ║
║  │                                                                         │ ║
║  │  [✓] Alertes par email          Seuil AQI: [120]                       │ ║
║  │  [✓] Notifications push         Fréquence: [Immédiat]                  │ ║
║  │  [ ] SMS d'urgence              Numéro: [+33 6 XX XX XX XX]             │ ║
║  │  [✓] Alertes météo              Seuil pluie: [80%]                     │ ║
║  │                                                                         │ ║
║  │  Types d'alertes:                                                       │ ║
║  │  [✓] Qualité air critique       [✓] Prédictions pics                   │ ║
║  │  [✓] Changements soudains       [ ] Alertes maintenance                │ ║
║  │                                                                         │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  [RETOUR] [TESTER ALERTES] [HISTORIQUE]                                    ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### Page Historique et Analytiques

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                          ANALYSES HISTORIQUES                             ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                        FILTRES ET PÉRIODE                             │ ║
║  │                                                                         │ ║
║  │  Période: [Semaine] [Mois] [3 Mois] [Année]   Type: [Tous] [AQI] [Temp] │ ║
║  │  Date début: [2024-01-01]   Date fin: [2024-01-15]                      │ ║
║  │  Agrégation: [Horaire] [Quotidienne] [Hebdomadaire]                    │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  ┌─────────────────────────────────────────────────────────────────────────┐ ║
║  │                          GRAPHIQUES ÉVOLUTION                          │ ║
║  │                                                                         │ ║
║  │          ▲                                                              │ ║
║  │        150 ──┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬─  │ ║
║  │             │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │   │ ║
║  │        100 ─┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼─  │ ║
║  │             │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │   │ ║
║  │         50 ─┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼──┼─  │ ║
║  │             │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │  │   │ ║
║  │          0 ─┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴─  │ ║
║  │        01 02 03 04 05 06 07 08 09 10 11 12 13 14 15                    │ ║
║  │                                                                         │ ║
║  │  [AQI] [Température] [Humidité] [PM2.5] [CO2] [Autres]                  │ ║
║  └─────────────────────────────────────────────────────────────────────────┘ ║
║                                                                            ║
║  ┌─────────────────────────────────┬───────────────────────────────────────┐ ║
║  │        STATISTIQUES             │        CORRÉLATIONS                   │ ║
║  │                                 │                                       │ ║
║  │  MOYENNE AQI: 78                │  Température ↔ Qualité Air: -0.45    │ ║
║  │  MAX AQI: 142                  │  Humidité ↔ Qualité Air: 0.32        │ ║
║  │  MIN AQI: 23                   │  Vent ↔ Qualité Air: -0.67            │ ║
║  │  ÉCART-TYPE: 24.5              │  Trafic ↔ Qualité Air: 0.78          │ ║
║  │                                 │                                       │ ║
║  │  JOURS BONNE QUALITÉ: 8/15     │  Précipitations ↔ Qualité Air: -0.23 │ ║
║  │  JOURS MAUVAISE QUALITÉ: 3/15  │                                       │ ║
║  │  ALERTES DÉCLENCHÉES: 12        │                                       │ ║
║  └─────────────────────────────────┴───────────────────────────────────────┘ ║
║                                                                            ║
║  [EXPORT CSV] [EXPORT PDF] [PARTAGER] [RETOUR]                             ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## 7. Architecture de Sécurité

```
ARCHITECTURE DE SÉCURITÉ IoT
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│                           COUCHES DE SÉCURITÉ                              │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   PHYSIQUE      │    │   RÉSEAU        │    │   APPLICATION   │         │
│  │   SÉCURITÉ      │    │   SÉCURITÉ      │    │   SÉCURITÉ      │         │
│  │                 │    │                 │    │                 │         │
│  │ • Châssis       │    │ • Firewall      │    │ • Authentif.    │         │
│  │ • Accès         │    │ • VPN           │    │ • Chiffrement   │         │
│  │ • Tamper        │    │ • IDS/IPS       │    │ • Autorisation  │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                    GESTION DES CLÉS ET CERTIFICATS                      │ │
│  │                                                                         │ │
│  │  • Certificats X.509 pour devices IoT                                  │ │
│  │  • Rotation automatique des clés                                       │ │
│  │  • Stockage sécurisé des secrets                                       │ │
│  │  • Authentification mutuelle TLS                                       │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   MONITORING    │    │   LOGGING       │    │   AUDIT         │         │
│  │   SÉCURITÉ      │    │   SÉCURITÉ      │    │   TRAIL         │         │
│  │                 │    │                 │    │                 │         │
│  │ • Détection     │    │ • Logs          │    │ • Traçabilité   │         │
│  │ • Alertes       │    │ • Intégrité     │    │ • Conformité    │         │
│  │ • Réponse       │    │ • Analyse       │    │ • Rapports      │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

*Ces diagrammes et maquettes constituent la base architecturale complète de votre système de surveillance de la qualité de l'air. Ils peuvent être utilisés directement dans votre rapport PFE ou adaptés selon vos besoins.*