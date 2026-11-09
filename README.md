\# Boutique Diayma — TD .NET n°1 (Environnement de développement)



\*\*Étudiante :\*\* Ramatoulaye Fall (RAFA) — ESP / UCAD, Département Génie Informatique

\*\*Chargé de cours :\*\* E. H. Ousmane Diallo

\*\*Environnement utilisé :\*\* Visual Studio Code + extension C# Dev Kit + CLI `dotnet` (Windows 11)

\*\*Dépôt d'origine :\*\* https://github.com/devehod/BoutiqueDiayma2025

\*\*Mon dépôt :\*\* https://github.com/Soxnarama/BoutiqueDiayma-RAFA

\*\*Exécutable Windows :\*\* \[LIEN GOOGLE DRIVE]



\---



\## 1. Récupération et exécution du code



```powershell

cd C:\\Users\\ramat

git clone https://github.com/devehod/BoutiqueDiayma2025

cd BoutiqueDiayma2025

dotnet restore

dotnet run --project P2FixAnAppDotNetCode\\Diayma.csproj

```



\- `git clone` télécharge le dépôt et son historique.

\- `dotnet restore` télécharge les paquets NuGet déclarés dans le `.csproj`.

\- `dotnet run` compile puis lance l'application sur \*\*http://localhost:62929\*\* (profil `P2FixAnAppDotNetCode` de `launchSettings.json`).



Les avertissements ⚠ affichés à la compilation (équivalent de la fenêtre d'avertissements de Visual Studio) signalent principalement que le framework ciblé, .NET Core 2.0, n'est plus pris en charge par Microsoft et que certains paquets NuGet de cette époque ont des vulnérabilités connues. Ce sont des avertissements, pas des erreurs : l'application compile et s'exécute.



\## 2. Projets de la solution



La solution `Diayma.sln` contient \*\*un seul projet\*\* :



| Projet | Fichier | Type |

|---|---|---|

| \*\*Diayma\*\* | `P2FixAnAppDotNetCode/Diayma.csproj` | Application web \*\*ASP.NET Core MVC\*\* (`Microsoft.NET.Sdk.Web`) |



Commande de vérification :

```powershell

dotnet sln list

```



Le dossier s'appelle `P2FixAnAppDotNetCode`, et c'est aussi le namespace racine du code, mais le projet s'appelle \*\*Diayma\*\*.



\## 3. Version du SDK .NET utilisée



Extrait de `Diayma.csproj` :

```xml

<Project Sdk="Microsoft.NET.Sdk.Web">

&#x20; <TargetFramework>netcoreapp2.0</TargetFramework>

&#x20; <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.6" />

```



\- Framework cible : \*\*.NET Core 2.0\*\* (`netcoreapp2.0`), avec ASP.NET Core 2.0 (`Microsoft.AspNetCore.All` 2.0.6).

\- SDK correspondant : \*\*.NET Core SDK 2.1.202\*\*, le SDK livré avec le runtime .NET Core 2.0.9.

\- Il n'y a pas de fichier `global.json` : aucune version de SDK n'est imposée.



Commande de vérification :

```powershell

Get-ChildItem -Recurse -Filter \*.csproj | Select-String "TargetFramework"

```



\## 4. Installation du SDK



Le SDK 2.1.202 (qui inclut le runtime .NET Core 2.0.9) se télécharge sur https://dotnet.microsoft.com/download/dotnet/2.0 (Windows x64). Il s'installe à côté du SDK 10 sans le remplacer.



```powershell

dotnet --list-sdks

dotnet --list-runtimes

```



On doit voir `2.1.202` dans la liste des SDK et `Microsoft.NETCore.App 2.0.9` dans les runtimes.



\## 5. Création de mon propre dépôt GitHub



Dépôt vide créé sur GitHub, puis :



```powershell

git remote rename origin prof

git remote add origin https://github.com/soxnarama/BoutiqueDiayma-RAFA.git

git branch -M main

git push -u origin main

```



\- `git remote rename origin prof` garde un lien vers le dépôt du professeur sous le nom `prof`.

\- `git remote add origin ...` fait de mon dépôt la destination par défaut.

\- `git push -u origin main` envoie le code et mémorise la branche de suivi.



\## 6. Bugs trouvés



\### Bug 1 — Le total du panier ignore les quantités



\*\*Reproduction :\*\* ajouter deux fois le même produit (ex. \*Clé USB 64GB\*, 1 000 l'unité). Dans le panier, la ligne affiche quantité 2 et sous-total 2 000, mais le \*\*Total\*\* affiche 1 000. Le résumé du panier en haut de page est faux de la même façon.



\*\*Cause :\*\* `Models/Cart.cs`, ligne 66 — on additionne le prix unitaire de chaque ligne sans le multiplier par la quantité :

```csharp

return GetCartLineList().Sum(x => x.Product.Price);

```

\*\*Correction :\*\*

```csharp

return GetCartLineList().Sum(x => x.Product.Price \* x.Quantity);

```

La moyenne (`GetAverageValue`, ligne 77) a le même défaut : elle fait la moyenne des prix des lignes sans tenir compte des quantités.



\### Bug 2 — Le stock ne diminue jamais après une commande



\*\*Reproduction :\*\* noter le stock d'un produit (ex. \*XIAOMI Redmi 13C\* : 10), l'ajouter au panier, valider la commande. De retour à l'accueil, le stock affiche toujours 10.



\*\*Cause :\*\* dans `Models/Repositories/ProductRepository.cs`, la liste est `static` (ligne 11), mais le constructeur la \*\*réinitialise à chaque création\*\* (lignes 15-16). Or le repository est enregistré en `AddTransient` dans `Startup.cs` (ligne 32) : un nouveau `ProductRepository` est créé à chaque requête, ce qui régénère les 9 produits avec leur stock d'origine et efface la mise à jour faite par la commande.



\*\*Correction :\*\* n'initialiser la liste qu'une seule fois :

```csharp

public ProductRepository()

{

&#x20;   if (\_products == null)

&#x20;   {

&#x20;       \_products = new List<Product>();

&#x20;       GenerateProductData();

&#x20;   }

}

```



\### Autres anomalies relevées



\- \*\*Langue espagnole inopérante :\*\* l'option \*Spanish\* existe dans le sélecteur, mais la culture `es` n'est pas dans `SupportedCultures` (`Startup.cs`, lignes 45-52) et il n'existe aucun fichier `.es.resx` ; l'interface reste en anglais.

\- \*\*Panier partagé entre tous les utilisateurs :\*\* le panier est enregistré en `AddSingleton` (`Startup.cs`, ligne 29). Un produit ajouté depuis un navigateur apparaît dans le panier d'un autre navigateur.

\- \*\*Devise incorrecte :\*\* les prix sont formatés avec `ToString("c")`, ce qui affiche £ ou € selon la langue au lieu du franc CFA.

\- \*\*Aucun contrôle de stock :\*\* on peut ajouter au panier plus d'unités que le stock disponible.



\## 7. Points d'arrêt



| Fichier | Ligne | Instruction | Méthode |

|---|---|---|---|

| `Components/CartSummaryViewComponent.cs` | 12 | `\_cart = cart as Cart;` | constructeur `CartSummaryViewComponent(ICart)` |

| `Controllers/ProductController.cs` | 15 | `\_productService = productService;` | constructeur `ProductController(...)` |

| `Controllers/OrderController.cs` | 17 | `\_cart = pCart;` | constructeur `OrderController(...)` |

| `Controllers/CartController.cs` | 15 | `\_cart = pCart;` | constructeur `CartController(...)` |

| `Startup.cs` | 20 | `Configuration = configuration;` | constructeur `Startup(IConfiguration)` |



\*\*Remarque :\*\* les cinq points d'arrêt sont tous dans des \*\*constructeurs\*\*. Ils montrent le moment où l'injection de dépendances d'ASP.NET Core crée chaque objet et lui fournit ses services (`ICart`, `IProductService`, etc.).



Seuls `Startup` (au démarrage), `ProductController` et `CartSummaryViewComponent` (à l'affichage de l'accueil) sont atteints avant l'affichage des produits. `CartController` n'est atteint qu'en ouvrant le panier ou en ajoutant un produit, et `OrderController` qu'en passant commande.



\## 8. Parcours d'exécution jusqu'à l'affichage des produits



Lancement en débogage avec \*\*F5\*\*. Modes utilisés : \*\*F11\*\* = pas à pas détaillé, \*\*F10\*\* = pas à pas principal, \*\*Shift+F11\*\* = pas à pas sortant.



\### Phase 1 — Démarrage du serveur (une seule fois)



| # | Namespace | Classe | Méthode | Mode et justification |

|---|---|---|---|---|

| 1 | `P2FixAnAppDotNetCode` | `Program` | `Main(string\[] args)` | Point d'entrée. \*\*F11\*\* sur `BuildWebHost(args).Run();` pour entrer dans `BuildWebHost`. |

| 2 | `P2FixAnAppDotNetCode` | `Program` | `BuildWebHost(string\[] args)` | `WebHost.CreateDefaultBuilder` et `Build` sont du code du framework : \*\*F10\*\*, car Just My Code empêche d'y entrer. |

| 3 | `P2FixAnAppDotNetCode` | `Startup` | `Startup(IConfiguration)` — \*\*point d'arrêt ligne 20\*\* | Appelé par le framework pendant `Build()`. \*\*F10\*\* pour exécuter l'affectation, puis \*\*Shift+F11\*\* pour sortir du constructeur. |

| 4 | `P2FixAnAppDotNetCode` | `Startup` | `ConfigureServices(IServiceCollection)` | Enregistrement des services dans le conteneur d'injection (panier en singleton, services et repositories en transient, localisation, MVC). \*\*F10\*\* ligne par ligne, inutile d'entrer dans les méthodes `Add...` du framework. |

| 5 | `P2FixAnAppDotNetCode` | `Startup` | `Configure(IApplicationBuilder, IHostingEnvironment)` | Construction du pipeline HTTP : fichiers statiques, localisation, session, et route par défaut `{controller=Product}/{action=Index}/{id?}`. \*\*F10\*\*. |

| 6 | `P2FixAnAppDotNetCode` | `Program` | `Main` → `.Run()` | Le serveur écoute sur `http://localhost:62929` et attend les requêtes. |



\### Phase 2 — Requête de la page d'accueil `/`



La route par défaut associe `/` à `ProductController.Index()`.



| # | Namespace | Classe | Méthode | Mode et justification |

|---|---|---|---|---|

| 7 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | constructeur, puis `GenerateProductData()` | Créé par l'injection de dépendances pour `ProductService`. Crée 9 objets `Product`. |

| 8 | `P2FixAnAppDotNetCode.Models` | `Product` | `Product(int, int, double, string, string)` ×9 | Constructeur appelé pour chaque produit du catalogue. |

| 9 | `P2FixAnAppDotNetCode.Models.Repositories` | `OrderRepository` | `OrderRepository()` | Deuxième dépendance de `ProductService`. |

| 10 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | `ProductService(IProductRepository, IOrderRepository)` | Reçoit les deux repositories. |

| 11 | `P2FixAnAppDotNetCode.Models.Services` | `LanguageService` | constructeur par défaut (implicite) | Singleton créé à la première utilisation ; aucun code visible. |

| 12 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `ProductController(IProductService, ILanguageService)` — \*\*point d'arrêt ligne 15\*\* | Le débogueur s'arrête ici. Les étapes 7 à 11 sont déjà exécutées : l'injection crée les dépendances \*\*avant\*\* d'appeler ce constructeur. Pour les observer, il faut poser des points d'arrêt supplémentaires dans leurs constructeurs. \*\*F10\*\* sur les lignes 15-16, puis \*\*Shift+F11\*\*. |

| 13 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `Index()` | Action appelée. \*\*F11\*\* sur la ligne 21 (`\_productService.GetAllProducts()`), car c'est là que les produits sont récupérés. |

| 14 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | `GetAllProducts()` | \*\*F11\*\* sur la ligne 29 pour descendre dans le repository. |

| 15 | `P2FixAnAppDotNetCode.Models.Repositories` | `ProductRepository` | `GetAllProducts()` | Filtre les produits en stock (`Where`) et les trie par nom (`OrderBy`). \*\*F10\*\* sur les appels LINQ (code du framework), puis \*\*Shift+F11\*\* pour remonter. |

| 16 | `P2FixAnAppDotNetCode.Models.Services` | `ProductService` | `GetAllProducts()` (retour) | Conversion `.ToList()`. \*\*Shift+F11\*\* pour revenir dans le contrôleur. |

| 17 | `P2FixAnAppDotNetCode.Controllers` | `ProductController` | `Index()` (ligne 22) | `return View(products);` : \*\*F10\*\*, le rendu de la vue est géré par le framework. |

| 18 | \*(vue Razor)\* | `Views/Product/Index.cshtml` | — | Génère le tableau HTML des produits (nom, description, prix, stock). |

| 19 | \*(vue Razor)\* | `Views/Shared/\_Layout.cshtml` | — | Gabarit commun ; appelle deux view components. |

| 20 | `P2FixAnAppDotNetCode.Components` | `CartSummaryViewComponent` | constructeur — \*\*point d'arrêt ligne 12\*\* | Appelé par `Component.InvokeAsync("CartSummary")` dans le layout. \*\*F10\*\*, puis \*\*Shift+F11\*\*. |

| 21 | `P2FixAnAppDotNetCode.Components` | `CartSummaryViewComponent` | `Invoke()` | Renvoie la vue du résumé du panier. |

| 22 | `P2FixAnAppDotNetCode.Components` | `LanguageSelectorViewComponent` | `Invoke(ILanguageService)` | Affiche le sélecteur de langue. |



Après l'étape 22, la page HTML complète est envoyée au navigateur et la liste des produits s'affiche.



\### Choix des modes de parcours



\- \*\*Pas à pas détaillé (F11)\*\* : utilisé quand la ligne appelle \*\*notre propre code\*\* dont on veut suivre le traitement (`GetAllProducts` du service, puis du repository).

\- \*\*Pas à pas principal (F10)\*\* : utilisé pour les appels au \*\*framework\*\* (`CreateDefaultBuilder`, `AddMvc`, LINQ, `View()`), dont le fonctionnement interne n'est pas l'objet de l'analyse.

\- \*\*Pas à pas sortant (Shift+F11)\*\* : utilisé pour \*\*remonter\*\* à l'appelant une fois la partie intéressante d'une méthode vue (sortie des constructeurs, retour du repository vers le service puis vers le contrôleur).



\## 9. Déploiement en exécutable Windows



Publication \*\*autonome\*\* (Self-Contained Deployment), conformément au profil `Properties/PublishProfiles/FolderProfile.pubxml` (`win-x64`, `SelfContained=true`) :



```powershell

cd C:\\Users\\ramat\\BoutiqueDiayma2025

dotnet publish P2FixAnAppDotNetCode\\Diayma.csproj -c Release -r win-x64 --self-contained true -o publish

cd publish

.\\Diayma.exe

```



\- `-c Release` : compilation optimisée.

\- `-r win-x64` : cible Windows 64 bits ; produit `Diayma.exe`.

\- `--self-contained true` : le runtime .NET Core 2.0 est embarqué ; l'exécutable fonctionne sur un PC sans .NET installé.

\- On lance l'exe \*\*depuis le dossier `publish`\*\* pour que l'application trouve son dossier `wwwroot` (CSS, images).

\- L'application est alors accessible sur \*\*http://localhost:5000\*\*.



\## 10. Lien vers l'exécutable



Le dossier `publish` a été compressé et déposé sur Google Drive :



```powershell

cd C:\\Users\\ramat\\BoutiqueDiayma2025

Compress-Archive -Path publish\\\* -DestinationPath Diayma-RAFA-win-x64.zip

```



\*\*Lien :\*\* \[LIEN GOOGLE DRIVE]



\## 11. Commits réalisés



```powershell

git log --oneline

```



| Commit | Contenu |

|---|---|

| \[à compléter] | |

