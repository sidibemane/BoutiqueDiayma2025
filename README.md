# Boutique Diayma

## Informations sur l'étudiant
- **Prénom :** Mané
- **Nom :** Sidibé
- **Classe :** Master2 GLSI
  

 ## 1. Projets de la solution

Après analyse de la solution, voici les projets identifiés :

- **P2FixAnAppDotNetCode** : Projet principal de l'application web ASP.NET Core MVC
  
  

## 2. Version SDK .NET utilisée

**D'aprés le fichier .csproj la version SDK .NET est :** .NET 10.0.100



## 3. Installation du SDK

Le SDK .NET a été installé depuis : https://dotnet.microsoft.com/download



## 4. Bugs identifiés

### Bug #1 : Runtime .NET manquant pour ASP.NET Core
- **Localisation :** Exécution de l’application (`dotnet run`)
- **Symptôme :** L’application ne démarre pas et affiche l’erreur suivante :  You must install or update .NET to run this application.
Framework: 'Microsoft.AspNetCore.App', version '6.0.0' (x64)
- **Cause probable :** Le runtime ASP.NET Core 6 n’était pas installé sur le système. Seul le runtime .NET 6 ou d’autres versions étaient présents.
- **Solution proposée :** Installer le runtime ASP.NET Core 6 (x64 .exe) via le site officiel Microsoft. Vérifier ensuite avec `dotnet --list-runtimes` pour confirmer la présence de `Microsoft.AspNetCore.App`.
- 
### Bug #2 : `UseMvc()` incompatible avec Endpoint Routing
- **Localisation :** `Startup.cs` → ligne 69 (`app.UseMvc(...)`)
- **Symptôme :** L’application démarre mais plante avec l’erreur suivante :Endpoint Routing does not support 'IApplicationBuilder.UseMvc(...)'.
To use 'IApplicationBuilder.UseMvc' set 'MvcOptions.EnableEndpointRouting = false' inside 'ConfigureServices(...)'.
- **Cause probable :** Depuis .NET Core 3.0 et .NET 6, Endpoint Routing est activé par défaut, et `UseMvc()` n’est plus compatible sans désactiver explicitement l’Endpoint Routing.
- **Solution proposée :**  
  1. **Solution rapide :** Désactiver l’Endpoint Routing dans `ConfigureServices` :  
  ```csharp
  services.AddMvc(options =>
  {
      options.EnableEndpointRouting = false;
  });
  


### 5. Analyse du flux d'exécution - Affichage des produits

### Points d'arrêt configurés :
1. **CartSummaryViewComponent** – Ligne 12  
   - Point d’arrêt placé sur la méthode `Invoke` pour suivre l’affichage du résumé du panier.
2. **ProductController** – Ligne 17  
   - Point d’arrêt placé sur l’action `Index` pour observer le chargement de la liste des produits.
3. **OrderController** – Ligne 20  
   - Point d’arrêt placé sur l’action principale (`Create` ou `Index`) pour suivre la création ou consultation des commandes.
4. **CartController** – Ligne 18  
   - Point d’arrêt placé sur l’action `Index` ou `AddToCart` pour observer l’ajout et l’affichage du panier.
5. **Startup** – Ligne 22  
   - Point d’arrêt placé dans `Configure` pour observer l’initialisation du pipeline HTTP, localisation et sessions.

## Flux d'exécution détaillé

### Étape 1 : Démarrage de l'application

**Namespace :** `BoutiqueDiayma`

**Classe :** `Startup`

**Méthode :** `Configure()`

**Ligne :** 20

**Description :**
- Configure le pipeline de requêtes HTTP
- Configure le routage MVC
- Met en place les middlewares nécessaires (routing, endpoints, authentication, etc.)
```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Product}/{action=Index}/{id?}");
});
```

---

### Étape 2 : Routage vers le contrôleur

**Namespace :** `BoutiqueDiayma.Controllers`

**Classe :** `ProductController`

**Méthode :** `Index()`

**Ligne :** 15

**Description :**
- Récupère la liste des produits depuis le service ou le repository
- Prépare les données pour la vue
- Retourne la vue avec les données des produits
```csharp
public IActionResult Index()
{
    var products = _productService.GetAllProducts();
    return View(products);
}
```

---

### Étape 3 : Affichage du panier (composant)

**Namespace :** `BoutiqueDiayma.ViewComponents`

**Classe :** `CartSummaryViewComponent`

**Méthode :** `Invoke()` ou `InvokeAsync()`

**Ligne :** 12

**Description :**
- Calcule le résumé du panier
- Affiche le nombre d'articles dans le panier
- Récupère les données du panier depuis la session ou le service
```csharp
public IViewComponentResult Invoke()
{
    var cartItemCount = _cartService.GetItemCount();
    return View(cartItemCount);
}
```

---

### Étape 4 : Rendu de la vue

**Fichiers concernés :**
- `Views/Product/Index.cshtml` : Affiche la liste des produits
- `Views/Shared/_Layout.cshtml` : Layout principal avec le composant CartSummary

**Description :**
- Le moteur Razor génère le HTML à partir des vues
- Le composant CartSummary est invoqué dans le layout
- Le HTML final est envoyé au navigateur

---

## Ordre d'exécution
```
```
1. Startup.Configure() 
    Configuration initiale du pipeline HTTP
   
2. ProductController.Index()
    Chargement des produits depuis la base de données
   
3. CartSummaryViewComponent.Invoke()
    Affichage du résumé du panier
   
4. Rendu de la vue
    Génération du HTML final et affichage dans le navigateur


## Modes de débogage

###  Pas à pas détaillé (F11 - Step Into)

**Quand l'utiliser :**
- Pour entrer dans chaque méthode et voir l'exécution interne
- Pour comprendre le flux complet ligne par ligne

**Utilisation dans ce projet :**
- Analyse approfondie de `Startup.Configure()` pour comprendre la configuration des middlewares
- Exploration du fonctionnement interne de `GetAllProducts()` pour voir les requêtes Entity Framework

**Exemple :**
```
Startup.Configure() [F11]
   app.UseRouting() [F11]
       Détails internes du routing...
```

---

### Pas à pas principal (F10 - Step Over)

**Quand l'utiliser :**
- Pour suivre ligne par ligne sans entrer dans les méthodes
- Pour rester dans le contexte de la méthode courante

**Utilisation dans ce projet :**
- Parcourir `ProductController.Index()` pour voir la séquence d'appels
- Suivre le flux sans se perdre dans les détails d'implémentation

**Exemple :**
```
var products = _productService.GetAllProducts(); [F10]
return View(products); [F10]
```

---

###  Pas à pas sortant (Shift+F11 - Step Out)

**Quand l'utiliser :**
- Pour sortir rapidement d'une méthode après avoir vu ce qui nous intéresse
- Pour remonter au niveau d'appel supérieur

**Utilisation dans ce projet :**
- Sortir rapidement de méthodes de logging ou de validation
- Revenir au contrôleur après avoir inspecté le repository

**Exemple :**
```
CartSummaryViewComponent.Invoke()
   _cartService.GetItemCount() [Shift+F11]
Retour à Invoke()

```

---



## 6. Déploiement

### Publication de l'exécutable Windows

**Commandes utilisées :**
```bash
dotnet publish -c Release -r win-x64 --self-contained true

**Emplacement de l'exécutable :**
`bin\Release\net6.0\win-x64\publish\`

```

---

## 7.Lien drive vers l'exécutable :

**[Télécharger Diayma.exe et fichiers associés]: https://drive.google.com/file/d/1dsECDuC6aiF0v6Zb5aneQpwKNJ7fQbvP/view?usp=sharing**

> ⚠️ Remarque pour le téléchargement :
> Google Drive peut afficher un avertissement "Ce fichier exécutable peut endommager votre ordinateur".  
> C’est normal pour les fichiers .exe. Ce fichier est sûr et provient de notre projet.  
> Pour l’exécuter :
> 1. Cliquez sur **Télécharger**.
> 2. Si le navigateur affiche un avertissement, choisissez **Conserver / Keep**.
> 3. Ensuite, double-cliquez sur `Diayma.exe` pour lancer l’application.









