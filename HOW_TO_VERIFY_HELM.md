# Comment vérifier que Helm a été utilisé dans le pipeline

## Méthode 1 : Vérifier les logs Jenkins

Dans les logs du pipeline Jenkins, vous devriez voir une ligne comme :
```
[Pipeline] sh
+ helm upgrade --install mon-app ./mon-app --namespace default --set image.repository=dockerraziza/new_app --set image.tag=latest
Release "mon-app" has been upgraded. Happy Helming!
```
ou
```
Release "mon-app" does not exist. Installing it now.
```

## Méthode 2 : Vérifier via kubectl (si vous avez accès au cluster)

### A. Vérifier les labels Helm sur les ressources
```bash
# Vérifier le deployment
kubectl get deployment mon-app -o yaml | grep -i helm

# Vérifier le service
kubectl get service mon-app-service -o yaml | grep -i helm
```

### B. Vérifier les annotations Helm
```bash
# Les ressources déployées par Helm ont ces annotations :
kubectl get deployment mon-app -o jsonpath='{.metadata.annotations.meta\.helm\.sh/release-name}'
# Devrait afficher : mon-app

kubectl get deployment mon-app -o jsonpath='{.metadata.annotations.meta\.helm\.sh/release-namespace}'
# Devrait afficher : default
```

### C. Vérifier les labels spécifiques à Helm
```bash
# Vérifier le label "managed-by"
kubectl get deployment mon-app -o jsonpath='{.metadata.labels.app\.kubernetes\.io/managed-by}'
# Devrait afficher : Helm

# Vérifier le label "chart"
kubectl get deployment mon-app -o jsonpath='{.metadata.labels.helm\.sh/chart}'
# Devrait afficher : mon-app-0.1.0
```

## Méthode 3 : Vérifier via Helm (sur Jenkins ou cluster Kubernetes)

Si vous avez accès à Helm sur votre serveur Jenkins ou cluster :

```bash
# Lister toutes les releases Helm
helm list

# Voir les détails d'une release
helm status mon-app

# Voir l'historique des déploiements
helm history mon-app

# Voir les valeurs utilisées
helm get values mon-app
```

## Méthode 4 : Comparer les noms des ressources

Les ressources déployées par Helm utilisent des noms générés :
- **Avec Helm** : `mon-app` (ou `mon-app-<release-name>`)
- **Sans Helm** : `mon-app-deployment` (nom exact du fichier YAML)

Vérifiez le nom du deployment :
```bash
kubectl get deployments
```

Si vous voyez `mon-app` au lieu de `mon-app-deployment`, c'est un bon signe que Helm a été utilisé.

## Méthode 5 : Vérifier la structure des labels

Les ressources déployées par Helm ont une structure de labels standardisée :

```bash
kubectl get deployment mon-app -o yaml
```

Recherchez ces labels :
- `app.kubernetes.io/name: mon-app`
- `app.kubernetes.io/instance: mon-app`
- `app.kubernetes.io/managed-by: Helm`
- `helm.sh/chart: mon-app-0.1.0`

## Indicateurs clés que Helm a été utilisé

✅ **OUI, Helm a été utilisé si :**
- Les logs Jenkins montrent `helm upgrade --install`
- Les ressources ont l'annotation `meta.helm.sh/release-name: mon-app`
- Les ressources ont le label `app.kubernetes.io/managed-by: Helm`
- La commande `helm list` montre la release `mon-app`
- Le nom du deployment est `mon-app` (pas `mon-app-deployment`)

❌ **NON, Helm n'a PAS été utilisé si :**
- Les ressources ont le nom exact du fichier YAML (`mon-app-deployment`)
- Pas d'annotations `meta.helm.sh/*`
- Pas de label `app.kubernetes.io/managed-by: Helm`
- Les logs Jenkins montrent `kubectl apply -f deployment.yaml`

## Commande rapide de vérification

Exécutez cette commande sur votre cluster Kubernetes :
```bash
kubectl get all -l app.kubernetes.io/managed-by=Helm
```

Si cela retourne des ressources, Helm a été utilisé !

