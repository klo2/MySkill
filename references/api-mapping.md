# Mapping détaillé API → outils MCP

Source : OpenAPI Service Contract Manager (déréférencé).

## 1. register — Créer un contrat

**Tool** : `contract_mcp-register`  
**Méthode** : POST `/api/v1/service-contracts`

**Body** (objet `body`) :
```json
{
  "contractId": "string",          // obligatoire côté client
  "goldenId": "string",
  "serviceId": "string",           // ex. "FLEX"
  "partner": {
    "id": "string",
    "label": "string"
  },
  "subscriberId": "string",
  "effectiveFrom": "YYYY-MM-DD",
  "effectiveTo": "YYYY-MM-DD",
  "targetAssets": [1001, 1002]     // array d'entiers (assetId)
}
```

Réponse typique : `ServiceContractResponse` (statut, version, équipements, etc.).

---

## 2. findById — Consulter un contrat

**Tool** : `contract_mcp-findById`  
**Path** : `contractId`

Retourne le contrat complet + historique des suspensions et avenants.

---

## 3. addTargetAsset — Ajouter un équipement

**Tool** : `contract_mcp-addTargetAsset`  
**Path** : `contractId`  
**Body** :
```json
{ "assetId": 1001 }
```

Crée automatiquement une `AssetServiceSubscription`.

---

## 4. removeTargetAsset — Retirer un équipement

**Tool** : `contract_mcp-removeTargetAsset`  
**Path** : `contractId` + `assetId`

Termine la souscription associée.

---

## 5. suspend — Suspendre

**Tool** : `contract_mcp-suspend`  
**Path** : `contractId`  
**Body** :
```json
{
  "startDate": "YYYY-MM-DD",   // optionnel (jour métier courant si absent)
  "endDate": "YYYY-MM-DD",     // optionnel
  "description": "string"
}
```

Propage la suspension aux souscriptions FLEX.

---

## 6. resume — Reprendre

**Tool** : `contract_mcp-resume`  
**Path** : `contractId`  
**Body** :
```json
{ "description": "string" }
```

---

## 7. renew — Renouveler

**Tool** : `contract_mcp-renew`  
**Path** : `contractId`  
**Body** :
```json
{
  "newEffectiveTo": "YYYY-MM-DD",
  "description": "string"
}
```

Crée un avenant.

---

## 8. terminate — Résilier

**Tool** : `contract_mcp-terminate`  
**Path** : `contractId`  
**Body** :
```json
{ "description": "string" }
```

Met la fin de validité à la veille (ou au début si pas encore démarré). Passe les souscriptions en `EXPIRED` / `TERMINATED`.

---

## 9. listForAsset — Souscriptions d'un équipement

**Tool** : `contract_mcp-listForAsset`  
**Path** : `assetId` (entier)  
**Query optionnels** : `type` (TELEMETRY|FLEX), `status` (ACTIVE|SUSPENDED|EXPIRED|TERMINATED)

Utile pour vérifier l'impact d'une action contrat sur un équipement précis.

---

## 10. export — Snapshot / reprise des souscriptions

**Tool** : `contract_mcp-export`  
**Query** :
- `cursor` : pagination (absent au premier appel)
- `updatedSince` : ISO-8601 (rattrapage incrémental)
- `limit` : 1–1000 (défaut 500)
- `type`, `status` : filtres

Retourne `items` + `nextCursor` + `snapshotAt`. Même format que les messages Pub/Sub.

---

## Statuts et transitions importantes

| Action | Statut avant | Statut après | Notes |
|--------|--------------|--------------|-------|
| register | — | ACTIVE (ou équivalent) | |
| suspend | ACTIVE | SUSPENDED | |
| resume | SUSPENDED | ACTIVE | |
| renew | ACTIVE / SUSPENDED | même statut, date fin mise à jour | |
| terminate | ACTIVE / SUSPENDED | TERMINATED / EXPIRED | irréversible |

Toujours faire un `findById` avant une transition pour éviter les 409.
