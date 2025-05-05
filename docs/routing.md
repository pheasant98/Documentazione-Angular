# Routing

In Angular, le routes (rotte) sono il meccanismo che consente di gestire la navigazione tra viste (routing) in una Single Page Application (SPA). Il modulo che gestisce questa funzionalità è il `RouterModule`.

## Impostare il routing

Per impostare il routing occore specificare qual’è l’URL di base da cui il router può comporre gli URL per tutte le view. L’URL di base è impostato nel file `index.html` con il tag `<base>` (`<base href="/">`) all'interno del tag `<head>`

Si specificano poi nel modulo le rotte attraverso l'oggetto Routes

!!! example
    ```ts
    // app-routing.module.ts
    const routes: Routes = [
        { path: '', component: HomeComponent },             // rotta base
        { path: 'about', component: AboutComponent },       // rotta /about
        { path : '** ', component : PageNotFoundComponent } // rotta wildcard,
    ];
    ```
Un oggetto Routes è un array di oggetti Routes che hanno se seguenti proprietà:

| Proprietà         | Tipo                          | Descrizione                                                                                                                                 |
|-------------------|-------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `path`            | `string`                      | Il percorso dell'URL che attiva la rotta (es. `'home'`, `'user/:id'`).                                                                    |
| `component`       | `Type<any>`                   | Il componente da visualizzare quando il percorso viene attivato.                                                                            |
| `redirectTo`      | `string`                      | Percorso verso cui reindirizzare in caso di corrispondenza.                                                                                 |
| `pathMatch`       | `'full'` \| `'prefix'`        | Specifica come confrontare il path: `'full'` per corrispondenza completa, `'prefix'` per parziale (default).                               |
| `children`        | `Route[]`                     | Array di rotte figlie per creare una struttura gerarchica (nested routes).                                                                  |
| `loadChildren`    | `string` \| `() => ...`       | Permette il lazy loading di un modulo figlio.                                                                                               |
| `canActivate`     | `any[]`                       | Array di guardie che controllano se la rotta può essere attivata.                                                                           |
| `canActivateChild`| `any[]`                       | Guardie applicate ai figli della rotta.                                                                                                     |
| `canDeactivate`   | `any[]`                       | Guardie che decidono se è possibile uscire dalla rotta corrente.                                                                            |
| `canLoad`         | `any[]`                       | Guardie che decidono se un modulo può essere caricato (usato con `loadChildren`).                                                           |
| `resolve`         | `{ [key: string]: any }`      | Oggetto che associa chiavi a resolver per caricare dati prima di attivare la rotta.                                                         |
| `data`            | `{ [key: string]: any }`      | Oggetto contenente dati statici (es. titolo, meta, flag) associati alla rotta.                                                              |
| `runGuardsAndResolvers` | `'paramsChange' \| 'pathParamsChange' \| 'always'` | Specifica quando ricaricare guardie e resolver se la rotta è già attiva. (solo per Angular 12)                                                                |
| `outlet`          | `string`                      | Nome dell’outlet dove caricare il componente (default: `'primary'`).                                                                        |

!!! note
    L'ordine delle rotte è importante perché il Router utilizza una strategia "prima corrispondenza vince" quando effettua il matching delle rotte. Per questo motivo, le rotte più specifiche devono essere elencate prima di quelle meno specifiche.

    Elenca quindi per prime:
    - le rotte con un percorso statico,
    - poi una rotta con percorso vuoto (''), che rappresenta la rotta di default,
    - e infine la wildcard route ('**'), che corrisponde a qualsiasi URL e viene selezionata solo se nessun'altra rotta corrisponde prima.


Una volta definite le rotte è necessario richiamare il metodo `forRoot()` (spiegato nel capitolo dedicato ai moduli).

!!! example
    ```ts
    // app-routing.module.ts

    const routes: Routes = [
        { path: '', component: HomeComponent },           
        { path: 'about', component: AboutComponent },
        { path : '** ', component : PageNotFoundComponent }    
    ];

    @NgModule({
        imports: [RouterModule.forRoot(routes)],
        exports: [RouterModule]
    })
    export class AppRoutingModule {}

    ```
Il modulo è configurato con le route ed esportato agli altri moduli dell’applicazione (Ad esempio nell'AppModule). Se si vuole definire delle sotto rotte in un altro modulo si dovrà utilizzare il metodo `forChild()`.
Infine nel file `index.html`, inserire la direttiva `<router-outlet>`. L’elemento specifica in quale posto del DOM vanno inserite le varie view gestite dal router.

## RouterLink

Un modo semplice per abilitare la navigazione tra le pagine in un'applicazione Angular è l'utilizzo della direttiva `RouterLink`. Quando applicata a un elemento del DOM, `RouterLink` trasforma l'elemento in un collegamento che, se cliccato, attiva la navigazione verso una specifica route definita nel router.
La navigazione risultante visualizzerà uno o più componenti instradati all’interno di uno o più elementi `<router-outlet>` presenti nella pagina.
!!! example
    ```ts
    <li class="nav-item">
        <a 
            class="nav-link" 
            routerLink="/tariffe" 
            routerLinkActive="active">
            Tariffe
        </a>
    </li>
    ```

## Path params

In Angular, per passara i path params basta aggiungere nella rotta ":" seguito dal nome del parametro.

!!! example
    ```ts
    // app-routing.module.ts
    ...
    const routes: Routes = [
        { path: '', component: HomeComponent },           
        { path: 'about', component: AboutComponent },
        { path: 'user/:id', component: UserComponent },
        { path : '** ', component : PageNotFoundComponent }    
    ];
    ...

    ```
Tramite `routerLink` bastera indicare il nome della rotta e il valore da comunicare: 
`<a [routerLink]="['/user', 42]">Vai a User 42</a>`, oppure tramite typescript `this.router.navigate(['/user', 42]);`.

per leggere i valori nel component occore utilizzare il service `ActivatedRoute`:
!!! example
    ```ts
    import { ActivatedRoute } from '@angular/router';

    constructor(private route: ActivatedRoute) {}

    ngOnInit() {
        this.route.paramMap.subscribe(params => {
            const id = params.get('id');
        });
    }

    ```

## Nesting routes

Le rotte annidate (nesting routes) in Angular permettono di organizzare l’applicazione inserendo componenti figli all’interno di un componente genitore, sfruttando il sistema di routing.
Quando l’app cresce in complessità, può essere utile definire rotte relative a componenti diversi da quello radice, per strutturare meglio la navigazione e la visualizzazione dei contenuti.

In questi casi, viene introdotto un secondo `<router-outlet>` all’interno del componente genitore, in aggiunta a quello già presente nell’`AppComponent`.

Ad esempio, supponiamo di avere due componenti figli: `child-a` e `child-b`.
Il componente `FirstComponent`, che funge da contenitore, include un proprio `<nav>` e un secondo `<router-outlet>`, nel quale verranno renderizzati dinamicamente i componenti figli in base alla rotta attiva.

!!! example
    ```html
    <h2>First Component</h2>
    <nav>
        <ul>
            <li><a routerLink="child-a">Child A</a></li>
            <li><a routerLink="child-b">Child B</a></li>
        </ul>
    </nav>
    <router-outlet />
    ```
Una rotta figlia è come qualsiasi altra rotta: ha bisogno sia di un percorso (path) che di un componente.
L’unica differenza è che le rotte figlie vengono definite all’interno di un array children all’interno della rotta padre.

!!! example
    ```ts
    const routes: Routes = [
        {
            path: 'first-component',
            component: FirstComponent,
            children: [
                {
                    path: 'child-a',
                    component: ChildAComponent,
                },
                {
                    path: 'child-b',
                    component: ChildBComponent,
                },
            ],
        },
    ];
    ```
## Guardie

Le guardie in Angular sono dei meccanismi di protezione che permettono di controllare l’accesso alle rotte della tua applicazione.
Le guardie eseguono controlli prima di attivare, disattivare, caricare o uscire da una rotta.

Di seguito viene riportato un esempio di guardia per controllare se lo user ha effettuato il login e quindi accedere alla dashboard:

!!! example
    ```ts
    @Injectable({
        providedIn: 'root'
    })
    export class AuthGuard implements CanActivate {
        constructor(private authService: AuthService, private router: Router) {}

        canActivate(): boolean {
            if (this.authService.isLoggedIn()) {
                return true;
            }
            this.router.navigate(['/login']);
            return false;
        }
    }

    ```
Viene implementata l'interfaccia `CanActivate` definendo il metodo `canActivate`.

!!! example
    ```ts
    const routes: Routes = [
        { path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] },
        { path: 'login', component: LoginComponent }
    ];
    ```
Utilizzando la proprietà dell'oggetto Route dedicata alle guardie più idonea (vedi tabella dedicata all'oggetto Route), viene aggiunta la guardia `AuthGuard`.

## Lazy loading

In Angular, è possibile configurare le rotte per caricare i moduli/componenti in modo lazy (caricamento a richiesta), il che significa che Angular carica i moduli/componenti solo quando necessario, invece di caricarli tutti all'avvio dell'applicazione.

Ogni rotta può caricare lazy il proprio modulo/componente stand-alone utilizzando loadComponent:
!!! example
    ```ts
    const routes: Routes = [
        {
            path: 'lazy',
            loadComponent: () => import('./lazy.component').then(c => c.LazyComponent)
        }
    ];
    ```
Con `loadComponent: () => import('./lazy.component')...` viene usato il lazy loading di un componente stand-alone. Angular non carica immediatamente `LazyComponent` all’avvio dell’applicazione, ma solo quando l’utente naviga a /lazy. Con `.then(c => c.LazyComponent)`, dopo l’importazione dinamica del file `lazy.component.ts`, si estrae il `LazyComponent` dal modulo importato.
