# Exports CSV du dashboard

Cette page décrit les fichiers générés par le bouton **Export CSV** du dashboard LoRaFlexSim. Les exports sont écrits dans un dossier horodaté de la forme `results/dashboard_exports/YYYY-MM-DD_HH-MM-SS/`.

Tous les CSV sont encodés en **UTF-8 avec BOM** (`utf-8-sig`) afin de s'ouvrir correctement dans Excel sous Windows 11. Le séparateur est la **virgule**.

## Ouverture sous Windows 11 / Excel

Méthode rapide :

1. Ouvrir l'Explorateur Windows.
2. Aller dans le dossier `results\dashboard_exports\...`.
3. Double-cliquer sur le fichier `.csv`.
4. Si les colonnes ne sont pas séparées correctement, utiliser **Données > À partir d'un fichier texte/CSV**, choisir l'encodage **65001: Unicode (UTF-8)** et le délimiteur **Virgule**.

Méthode PowerShell utile pour trouver le dernier export :

```powershell
Get-ChildItem .\results\dashboard_exports -Directory |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1
```

## Chargement générique avec pandas

```python
from pathlib import Path
import pandas as pd

export_dir = Path("results/dashboard_exports/2026-06-25_12-00-00")
raw_packets = pd.read_csv(export_dir / "raw_packets.csv", encoding="utf-8-sig")
```

Pour charger automatiquement tous les CSV disponibles :

```python
from pathlib import Path
import pandas as pd

export_dir = Path("results/dashboard_exports/2026-06-25_12-00-00")
csv_tables = {
    path.stem: pd.read_csv(path, encoding="utf-8-sig")
    for path in export_dir.glob("*.csv")
}
```

## `raw_packets.csv`

- **Quand il est généré** : lorsque l'export contient des événements de paquets (`runs_events`). Le dashboard collecte ces événements à la fin de la simulation ou au moment de l'export si la simulation est terminée.
- **Ce qu'il contient** : une ligne par événement paquet exporté, avec les colonnes normalisées en tête puis les colonnes brutes disponibles dans le journal d'événements.
- **Principales colonnes** :
  - `time` : temps de simulation en secondes. Si la colonne brute `time` n'existe pas, elle est dérivée de `start_time`.
  - `node_id` : identifiant du nœud émetteur.
  - `sf` : spreading factor utilisé, généralement entre 7 et 12.
  - `tx_ok` : indicateur d'émission, toujours `1` pour une ligne paquet exportée.
  - `rx_ok` : `1` si `result == "Success"`, sinon `0`.
  - `payload_bytes` : taille de charge utile, prise depuis la configuration du run si disponible.
  - `run` : identifiant du run, `1` par défaut si absent dans les événements.
  - Colonnes additionnelles possibles : `start_time`, `result`, canal, passerelle, RSSI/SNR/SNIR ou autres champs d'événement selon le simulateur.
- **Excel** : ouvrir le CSV en UTF-8 avec délimiteur virgule. Filtrer `rx_ok` pour isoler les transmissions reçues, ou créer un tableau croisé dynamique par `run`, `node_id` et `sf`.
- **pandas** :

```python
packets = pd.read_csv(export_dir / "raw_packets.csv", encoding="utf-8-sig")
pdr_by_run = packets.groupby("run")["rx_ok"].mean()
pdr_by_node = packets.groupby(["run", "node_id"])["rx_ok"].mean()
sf_success = packets.groupby(["sf", "rx_ok"]).size().unstack(fill_value=0)
```

## `metrics_complete.csv`

- **Quand il est généré** : lorsque l'export contient des métriques agrégées (`runs_metrics`).
- **Ce qu'il contient** : une ligne par run avec toutes les métriques retournées par le simulateur, aplaties par `pandas.json_normalize` lorsque des dictionnaires imbriqués existent.
- **Principales colonnes** :
  - `run` : identifiant du run.
  - `PDR`, `tx_attempted`, `delivered`, `collisions`, `collisions_snir`, `duplicates` : métriques de livraison et de pertes.
  - `energy_J`, `energy_nodes_J`, `energy_gateways_J` : énergie totale et par famille d'équipement.
  - `avg_delay_s`, `avg_arrival_interval_s`, `throughput_bps` : délai, intervalle moyen et débit.
  - `retransmissions`, `ack_success_count`, `ack_nack_count`, `ack_total_count`, `ack_success_rate`, `ack_nack_rate` : indicateurs ADR/ACK quand disponibles.
  - `simulation_duration_s` : durée de simulation ; si elle est absente des métriques, elle est déduite du temps maximal dans `raw_packets.csv`.
  - Colonnes aplaties possibles : `pdr_by_node.*`, `recent_pdr_by_node.*`, `pdr_by_gateway.*`, `energy_by_node.*`, `qos_refresh_benchmark.*`, `runtime_profile_s.*`, etc.
- **Excel** : utiliser ce fichier comme table principale de comparaison entre runs. Les colonnes aplaties contenant des points (`.`) restent des colonnes Excel normales.
- **pandas** :

```python
metrics = pd.read_csv(export_dir / "metrics_complete.csv", encoding="utf-8-sig")
summary = metrics[["run", "PDR", "energy_J", "throughput_bps", "simulation_duration_s"]]
correlation = metrics[["PDR", "energy_J", "throughput_bps"]].corr(numeric_only=True)
```

## `energy_summary.csv`

- **Quand il est généré** : lorsque `metrics_complete.csv` est généré, donc en présence de `runs_metrics`.
- **Ce qu'il contient** : une synthèse compacte de l'énergie par run.
- **Principales colonnes** :
  - `run` : identifiant du run.
  - `total_energy_joule` : valeur de `energy_J`.
  - `energy_nodes_joule` : valeur de `energy_nodes_J`.
  - `energy_gateways_joule` : valeur de `energy_gateways_J`.
  - `sim_duration_s` : durée du run en secondes.
- **Excel** : idéal pour tracer un histogramme `total_energy_joule` par `run` ou comparer la part nœuds/passerelles avec un graphique empilé.
- **pandas** :

```python
energy = pd.read_csv(export_dir / "energy_summary.csv", encoding="utf-8-sig")
energy["energy_per_second"] = energy["total_energy_joule"] / energy["sim_duration_s"].replace(0, pd.NA)
energy_by_run = energy.set_index("run")
```

## `nodes_metrics.csv`

- **Quand il est généré** : lorsque des métriques par nœud existent dans `runs_metrics`.
- **Ce qu'il contient** : une ligne par couple `(run, node_id)`.
- **Principales colonnes** :
  - `run`, `node_id` : identifiants.
  - `pdr`, `recent_pdr` : PDR global et PDR récent du nœud.
  - `energy_j`, `airtime_s` : énergie et temps d'antenne du nœud.
  - `energy_tx_j`, `energy_rx_j`, `energy_sleep_j`, `energy_listen_j` : ventilation énergétique standard.
  - Colonnes additionnelles possibles : `energy_<mode>_j` si le simulateur expose d'autres états d'énergie.
- **Excel** : créer un tableau croisé par `run` et `node_id`, puis appliquer une mise en forme conditionnelle sur `pdr` ou `energy_j`.
- **pandas** :

```python
nodes = pd.read_csv(export_dir / "nodes_metrics.csv", encoding="utf-8-sig")
worst_nodes = nodes.sort_values(["run", "pdr"]).groupby("run").head(5)
energy_stats = nodes.groupby("run")["energy_j"].describe()
```

## `gateways_metrics.csv`

- **Quand il est généré** : lorsque des métriques par passerelle existent dans `runs_metrics`.
- **Ce qu'il contient** : une ligne par couple `(run, gateway_id)`.
- **Principales colonnes** :
  - `run`, `gateway_id` : identifiants.
  - `pdr` : PDR observé côté passerelle.
  - `energy_j` : énergie de la passerelle.
  - `energy_tx_j`, `energy_rx_j`, `energy_sleep_j`, `energy_listen_j` : ventilation énergétique standard.
  - Colonnes additionnelles possibles : `energy_<mode>_j` pour des états énergétiques spécifiques.
- **Excel** : filtrer par `gateway_id` pour comparer les passerelles ou faire un graphique `energy_j` par passerelle.
- **pandas** :

```python
gateways = pd.read_csv(export_dir / "gateways_metrics.csv", encoding="utf-8-sig")
gateway_balance = gateways.pivot_table(index="run", columns="gateway_id", values="pdr")
energy_by_gateway = gateways.groupby("gateway_id")["energy_j"].mean()
```

## `sf_distribution.csv`

- **Quand il est généré** : lorsque `runs_metrics` contient `sf_distribution`.
- **Ce qu'il contient** : distribution normalisée des spreading factors par run, une ligne par SF présent dans la distribution.
- **Principales colonnes** :
  - `run` : identifiant du run.
  - `sf` : spreading factor.
  - `node_count` : nombre de nœuds utilisant ce SF.
- **Excel** : insérer un graphique en colonnes avec `sf` en axe horizontal et `node_count` en valeurs, éventuellement segmenté par `run`.
- **pandas** :

```python
sf = pd.read_csv(export_dir / "sf_distribution.csv", encoding="utf-8-sig")
sf_pivot = sf.pivot_table(index="sf", columns="run", values="node_count", fill_value=0)
avg_sf = (sf["sf"] * sf["node_count"]).groupby(sf["run"]).sum() / sf.groupby("run")["node_count"].sum()
```

## `tx_power_distribution.csv`

- **Quand il est généré** : lorsque `runs_metrics` contient `tx_power_distribution`.
- **Ce qu'il contient** : distribution normalisée des puissances d'émission par run.
- **Principales colonnes** :
  - `run` : identifiant du run.
  - `tx_power_dbm` : puissance d'émission en dBm.
  - `node_count` : nombre de nœuds utilisant cette puissance.
- **Excel** : créer un graphique en colonnes `tx_power_dbm` / `node_count` pour visualiser les réglages ADR ou fixes.
- **pandas** :

```python
tx = pd.read_csv(export_dir / "tx_power_distribution.csv", encoding="utf-8-sig")
tx_pivot = tx.pivot_table(index="tx_power_dbm", columns="run", values="node_count", fill_value=0)
avg_tx_power = (tx["tx_power_dbm"] * tx["node_count"]).groupby(tx["run"]).sum() / tx.groupby("run")["node_count"].sum()
```

## `qos_clusters_metrics.csv`

- **Quand il est généré** : lorsque les métriques QoS par cluster existent dans `runs_metrics`. Le fichier est quand même écrit avec l'en-tête lorsque les métriques globales existent, mais il peut être vide si aucun cluster QoS n'a été configuré.
- **Ce qu'il contient** : une ligne par couple `(run, cluster_id)`.
- **Principales colonnes** :
  - `run`, `cluster_id` : identifiants.
  - `pdr` : PDR du cluster.
  - `pdr_target` : objectif de PDR du cluster.
  - `pdr_gap` : écart entre le PDR observé et la cible.
  - `throughput_bps` : débit du cluster.
  - `node_count` : nombre de nœuds dans le cluster.
  - `sf_channel` : charge JSON compacte décrivant la répartition SF/canal du cluster lorsque disponible.
- **Excel** : filtrer `pdr_gap < 0` pour trouver les clusters sous objectif. La colonne `sf_channel` peut être conservée comme texte.
- **pandas** :

```python
qos = pd.read_csv(export_dir / "qos_clusters_metrics.csv", encoding="utf-8-sig")
under_target = qos[qos["pdr_gap"] < 0]
cluster_summary = qos.groupby("cluster_id")[["pdr", "throughput_bps", "node_count"]].mean(numeric_only=True)
```

## `runs_config.csv`

- **Quand il est généré** : lorsque le dashboard dispose des configurations de runs (`runs_configs`).
- **Ce qu'il contient** : une ligne par run, avec la configuration aplatie par `pandas.json_normalize`.
- **Principales colonnes** :
  - Colonnes garanties en tête si absentes du payload : `run`, `seed`, `traffic.mode`, `traffic.packet_interval_s`, `traffic.first_packet_interval_s`, `traffic.packets_per_node`, `traffic.payload_size_bytes`, `radio.phy_model`, `radio.fixed_sf`, `radio.fixed_tx_power_dbm`, `radio.num_channels`, `topology.num_nodes`, `topology.num_gateways`, `topology.area_size_m`.
  - Colonnes additionnelles possibles : tous les paramètres imbriqués présents dans la configuration du run, aplatis avec des points.
- **Excel** : utiliser ce fichier pour vérifier les paramètres de chaque run et le joindre mentalement ou via Power Query avec `metrics_complete.csv` sur la colonne `run`.
- **pandas** :

```python
configs = pd.read_csv(export_dir / "runs_config.csv", encoding="utf-8-sig")
metrics = pd.read_csv(export_dir / "metrics_complete.csv", encoding="utf-8-sig")
joined = metrics.merge(configs, on="run", how="left", suffixes=("", "_config"))
pdr_by_payload = joined.groupby("traffic.payload_size_bytes")["PDR"].mean()
```

## `run_X_config.json`

- **Quand il est généré** : pour chaque entrée de `runs_configs`, en même temps que `runs_config.csv`. `X` correspond à l'ordre d'export (`run_1_config.json`, `run_2_config.json`, etc.).
- **Ce qu'il contient** : la configuration complète d'un run au format JSON, indentée, triée par clé et encodée en UTF-8. Ce fichier est plus fidèle que `runs_config.csv` pour relire les structures imbriquées.
- **Principales clés** :
  - `run`, `seed` : identifiants et graine.
  - `traffic` : mode de trafic, intervalles, nombre de paquets, taille de payload.
  - `radio` : modèle PHY, SF fixe, puissance fixe, nombre de canaux, options SNIR/ADR selon le scénario.
  - `topology` : nombre de nœuds, passerelles et taille de zone.
  - Autres sections possibles selon les options activées dans le dashboard.
- **Windows / Excel** : Excel peut importer le JSON via **Données > Obtenir des données > À partir d'un fichier > À partir de JSON**. Pour une lecture rapide, ouvrir le fichier avec Visual Studio Code, Notepad++ ou le Bloc-notes Windows.
- **pandas** :

```python
import json
from pathlib import Path
import pandas as pd

config_path = export_dir / "run_1_config.json"
with config_path.open(encoding="utf-8") as fh:
    config = json.load(fh)

flat_config = pd.json_normalize(config)
```

## Conseils d'analyse croisée

- Joindre `metrics_complete.csv` et `runs_config.csv` sur `run` pour expliquer les performances par les paramètres.
- Utiliser `raw_packets.csv` pour les analyses temporelles fines et `metrics_complete.csv` pour les comparaisons synthétiques.
- Utiliser `nodes_metrics.csv` et `gateways_metrics.csv` pour détecter les déséquilibres locaux.
- Utiliser `sf_distribution.csv`, `tx_power_distribution.csv` et `qos_clusters_metrics.csv` pour diagnostiquer les effets ADR/QoS.
