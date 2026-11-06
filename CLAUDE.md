# Xdiag

Application Windows desktop qui **diagnostique** pourquoi un laptop gaming de 2-4 ans a perdu en performances, puis propose des remédiations explicitées. Pas de "booster" générique : uniquement des causes réelles et mesurables.

## Machine de référence (à supporter en priorité)
- CPU : AMD Ryzen 5 4600H (6C/12T)
- GPU : NVIDIA RTX 3050 Laptop 4 Go
- OS : Windows 10/11
- Symptôme : 120 FPS -> 70-80 FPS sur Fortnite et Rainbow Six Siege en 2 ans, sans changement matériel.

## Stack
- C# / .NET 8, WPF (pas WinUI 3 pour le MVP)
- LibreHardwareMonitorLib (NuGet) pour températures, fréquences, charges
- WMI/CIM via System.Management pour RAM, slots, GPU, plan d'alimentation, version Windows
- N'utilise pas `wmic` (déprécié sous Windows 11) : CIM/WMI ou PowerShell
- Tests : xUnit
- Les capteurs demandent les droits administrateur : manifeste `requireAdministrator` sur l'app

## Architecture (3 couches, solution modulaire)
1. `Xdiag.Collector` : sondes passives, AUCUNE modification système. Interfaces + implémentations Windows + fausses implémentations pour les tests.
2. `Xdiag.Diagnostics` : moteur de règles. Une règle = données d'entrée -> seuil -> constat -> explication -> remédiation suggérée. Chaque Finding doit être traçable (donnée source, valeur, seuil, pourquoi).
3. `Xdiag.App` (WPF) : tableau de bord feux vert/orange/rouge par pilier, détail par pilier, MVVM.
4. `Xdiag.Remediation` : Phase 2+ seulement. Confirmation utilisateur obligatoire + rollback pour chaque action.
5. `Xdiag.Tests`

## Les 6 piliers
1. Thermique et throttling : température + fréquence corrélées (CPU > 95°C avec fréquence en chute, GPU > 87°C), ventilateurs à fond mais températures hautes = poussière/pâte.
2. GPU et pilotes : version pilote NVIDIA, jeux sur dGPU et non iGPU, gestion d'alimentation NVIDIA, Reflex, HAGS.
3. CPU et charge système : goulot CPU (CPU ~100 %, GPU sous-utilisé), processus de fond gourmands.
4. RAM : single vs dual channel (critique sur Ryzen), quantité (16 Go min), saturation (> 90 %), fréquence réelle.
5. SSD : SMART, température NVMe (> 93-97°C), TRIM, espace libre (< 15 % = alerte).
6. OS et jeux : plan d'alimentation, HAGS, Game Mode, Xbox Game Bar, overlays.

## Roadmap
- **Phase 1 (EN COURS) : MVP diagnostic en lecture seule.** Monitoring temps réel, specs complètes, rapport statique basé sur seuils. Ne rien modifier sur le système.
- Phase 2 : actions simples (plan d'alimentation, HAGS, Game Mode, Game Bar, GPU dédié par jeu, TRIM).
- Phase 3 : assistant DDU, presets de jeu, profils, mode concentration.
- Phase 4 : base de pilotes stables, rapports communautaires, tutoriels.

Ne saute aucune phase. Ne commence pas la Phase 2 sans mon feu vert.

## Principes
- Transparence : chaque alerte affiche donnée source, seuil et impact attendu.
- Sécurité : aucune modification système sans confirmation explicite ; rollback pour chaque action.
- Pas de promesses non fondées, pas de "nettoyage de registre".
- Seuils calibrés pour CPU/GPU mobiles.
- Tolérer les données manquantes (SMART, capteurs NVMe variables selon fabricant) : afficher "indisponible", ne jamais planter.
- Extensible : ajouter un pilier ou une règle ne doit pas toucher au reste.

## Conventions
- Code et identifiants en anglais, textes UI et explications de diagnostic en français.
- Les seuils de règles sont dans des fichiers de config/JSON, pas en dur.
- Commits petits et fréquents, un jalon = un commit.
- Après chaque étape : `dotnet build` et `dotnet test` doivent passer.

## Critère de succès
Sur la machine de référence, l'app identifie la/les causes racines de la chute de FPS par corrélation de données, guide vers une remédiation vérifiable, ne fait prendre aucun risque sans explication, et donne un tableau de bord comparable dans le temps (historique des rapports).
