# Services

In Angular, un servizio è una classe che contiene logica riutilizzabile e funzionalità condivise tra più componenti o moduli. Permette di organizzare il codice, separando le responsabilità: ad esempio, anziché scrivere la logica di accesso ai dati direttamente in un componente, si crea un servizio dedicato, e il componente lo utilizza quando necessario, rendendo così l’applicazione più flessibile inserendo un provider dedicato all'accesso dati.

Un servizio può occuparsi di per esempio: chiamate HTTP a un'API, gestione dello stato dell'applicazione, operazioni matematiche, accesso a dati locali, formattazioni, e così via. Un servizio è una classe TypeScript annotata con il decoratore `@Injectable()`.

## Dependency Injection

Per usare un servizio in un componente, Angular usa un meccanismo chiamato Dependency Injection. 

La Dependency Injection è un meccanismo che fornisce automaticamente le dipendenze richieste da una classe, senza che sia la classe stessa a crearle. Quindi un oggetto (consumer) delega il compito di fornire le dipendenze necessarie a del codice esterno (injector). In Angular, le dipendenze sono in genere servizi, ma possono essere anche altri oggetti o classi.

<figure markdown>
  ![component6](./immagini/injector-injects.png){ width="400" : .center}
  <figcaption>Figura 6: Schema funzionamento DI</figcaption>
</figure>
Se un componente ha bisogno di un servizio, Angular lo "inietta" nel costruttore del componente:

!!! example
    ```ts
    @Component({...})
    export class ProfiloComponent implements OnInit {
        constructor(private datiUtenteService: DatiUtenteService) {}

        ngOnInit(): void {
            const nome = this.datiUtenteService.getNomeUtente();    
        }
    }

    ```

In questo esempio, Angular crea un'istanza di `DatiUtenteService` e la passa al costruttore di `ProfiloComponent` automaticamente. Il componente non si preoccupa di come viene creato il servizio, si limita a usarlo. Perciò, un componente che vuole utilizzare il servizio inserisce la sua dichiarazione nel costruttore.
E’ fondamentale annotare il parametro con il tipo corretto che sarà la chiave usata dall’injector per capire che istanza iniettare.

## Injectables
Come scritto precedentemente, i servizi possono essere iniettati in un componente come dipendenze, e un servizio è una classe che utilizza il decoratore `@Injectable`. Inoltre un servizio puo avere sua volta delle dipendenze.

Nel seguente esempio viene mostrato come creare un servizio che a sua volta ha una dipendenza, risolta tramite l'injection. 
!!! example
    ```ts
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';

    @Injectable({
    providedIn: 'root'
    })
    export class UtenteService {
        constructor(private http: HttpClient) {}

        getUtenti() {
            return this.http.get('/api/utenti');
        }
    }

    ```

Nell'esempio `@Injectable` contiene dei metadati. Infatti, la proprietà `providedIn` indica a Angular dove registrare il servizio nel sistema di iniezione delle dipendenze. 
Di seguito i valori che puo assumere:
- 'root': È l'opzione più comune e consigliata. Significa che il servizio sarà disponibile  ovunque nell’applicazione, quindi creato una sola volta (singleton) e sarà caricato automaticamente da Angular, senza doverlo registrare manualmente nel modulo.
- 'any': Questa opzione crea istanze separate del servizio in ogni modulo che lo richiede, utile per servizi che non devono condividere stato tra moduli.
- 'SomeModule' : permette di indicare un modulo specifico in cui fornire il servizio. Questo è utile quando vuoi limitare l’ambito del servizio a un modulo particolare (come un modulo lazy-loaded). Nel providedIn deve essere indicato il nome del modulo, per esempio:
`providedIn: MyFeatureModule`

## Providers

Nel capitolo dedicato ai componenti, abbiamo visto che il decoratore `@Component` include una proprietà chiamata `providers`. In Angular, i providers rappresentano il meccanismo attraverso cui viene definito come fornire i servizi (dependencies) all’interno dell’applicazione, utilizzando il sistema di Dependency Injection. Quando un servizio viene registrato come provider, si comunica ad Angular come creare o recuperare un'istanza di quel servizio. Sarà poi compito dell'Injector mantenere e gestire la mappa tra i token (le chiavi) e le istanze associate.

Abbiamo già incontrato la registrazione tramite `@Injectable({ providedIn: 'root' })`, che rende il servizio un singleton a livello globale, condiviso da tutta l’applicazione. Tuttavia, è anche possibile registrare un servizio direttamente all’interno della proprietà providers di un componente: in questo modo il servizio è scoped al componente stesso, ovvero vive solo finché vive il componente, e ogni istanza del componente riceve una propria istanza del servizio.

!!! example
    ```ts
    @Component({
        ....
        providers: [LoggerService]  // provider locale al componente
        })
        export class ChildComponent {
        constructor(private logger: LoggerService) {}

        logMessage() {
            this.logger.log('Messaggio dal ChildComponent');
        }

    }
    ```
In questo esempio, Angular crea un’istanza locale di `LoggerService` per ogni istanza del componente. L'istanza viene creata tramite il costruttore della classe, sfruttando implicitamente la strategia `useClass`, in cui si specifica solo la chiave (`LoggerService`) e Angular deduce la classe da instanziare.

!!! example
    ```ts
    @Component({
        ....
        providers: [{ provide: LoggerService, useClass: LoggerService }]
        })
    ```
La configurazione estesa di un provider è un oggetto letterale con due proprietà:
- La proprietà provide contiene il token, che funge da chiave per richiedere il valore della dipendenza.
- La seconda proprietà è un oggetto di definizione del provider, che indica all’injector come creare il valore della dipendenza. La definizione del provider sarà tratta nelle prossime sezioni.

### useClass
Permette di fornire una classe alternativa quando viene richiesto un certo token. useClass consente di creare e restituire una nuova istanza della classe specificata.

!!! example
    ```ts
    @Component({
        ....
        providers: [{ provide: MyService, useClass: MockService }]
        })
       
    ```
In questo caso, qualsiasi classe che richieda `MyService` riceverà in realtà un’istanza di `MockService`.

Per rendere il servizio disponibile in tutta l’applicazione, lo stesso provider può essere registrato direttamente nella definizione del modulo root. In questo modo sarà usata una sola istanza di `MockService` per tutti i componenti del modulo.

### useExisting

useExisting consente di collegare un token a un altro, creando un alias. In questo modo, il primo token diventa un riferimento alternativo al servizio associato al secondo token, permettendo di accedere allo stesso oggetto tramite due nomi diversi.

!!! example
    ```ts
    @Component({
        ....
        providers: [ NewLogger, { provide: OldLogger, useExisting: NewLogger }]
        })
       
    ```
Nell’esempio precedente, l’injector fornisce la stessa istanza singleton di `NewLogger` quando un componente richiede sia `NewLogger` sia `OldLogger`. In questo modo, `OldLogger` è un alias di `NewLogger`.

!!!warning
    Non utilizzare `useClass` per creare un alias tra `OldLogger` e `NewLogger`, perché questo comporterebbe la creazione di due istanze distinte di `NewLogger`.

### useFactory

useFactory permette di generare un oggetto dipendenza attraverso una funzione factory. Questo approccio è ideale quando è necessario creare valori dinamici, basati su dati disponibili nel sistema di Dependency Injection o provenienti da altre parti dell'applicazione.

Nell’esempio seguente, si crea un servizio `ApiService` che utilizza un valore di configurazione dinamico (apiUrl) fornito da un `ConfigService`.

!!! example
    ```ts
    //config.service.ts
    import { Injectable } from '@angular/core';

    @Injectable()
    export class ConfigService {
        private environment = 'production';
        private configs = {
            development: { apiUrl: 'http://localhost:3000' },
            production: { apiUrl: 'https://api.example.com' }
        };

        get apiUrl(): string {
            return this.configs[this.environment].apiUrl;
        }
    } 
    ```

    ```ts
    //api.service.ts
    export class ApiService {
        constructor(private apiUrl: string) {}

        getEndpoint(): string {
            return `${this.apiUrl}/endpoint`;
        }
    }
    ```
In base all'ambiente viene fornito un url differente da `ConfigService`.
Per utilizzare il useFactory:

!!! example
    ```ts
    //api.service.provider.ts
    import { ApiService } from './api.service';
    import { ConfigService } from './config.service';

    export function apiServiceFactory(config: ConfigService): ApiService {
        return new ApiService(config.apiUrl);
    }

    export const apiServiceProvider = {
        provide: ApiService,
        useFactory: apiServiceFactory,
        deps: [ConfigService]
    };
    ```

 Il campo useFactory specifica che il provider è una funzione factory implementata da `apiServiceFactory`. La funzione factory `apiServiceFactory` accede a `ApiService`. La proprietà deps è un array di token di provider. Angular risolve questi token e inietta i servizi corrispondenti nei parametri della funzione factory, nell’ordine specificato.

!!! example
    ```ts
    @Component({
        ...
        providers: [ConfigService, apiServiceProvider]
    })
    export class AppComponent {
     endpoint: string;

        constructor(private apiService: ApiService) {
            this.endpoint = this.apiService.getEndpoint();
        }
    }
    ```  
Nel component dove viene utilizzato `ApiService`, vengono indicati nel providers il provider factory `apiServiceProvider` e la sua dipendenza `ConfigService`.

### useValue

useValue consente di associare un valore statico a un token di Dependency Injection.
Questa tecnica è particolarmente utile per fornire costanti di configurazione a runtime, come URL di base, flag di funzionalità o valori predefiniti. Inoltre, è molto usata nei test unitari per sostituire servizi reali con dati mock, semplificando la scrittura e il controllo dei test.
Per utilizzare useValue occore usare l'oggetto `InjectionToken` come token del provider per le dipendenze che non sono classi.

!!! example
    ```ts
    //app.config.ts
    import { InjectionToken } from '@angular/core';

    export interface AppConfig {
        title: string;
    }

    export const APP_CONFIG = new InjectionToken<AppConfig>('Descrizione del token app.config');

    ``` 

Nell’esempio viene definito un token `APP_CONFIG` di tipo `InjectionToken`. Il parametro di tipo opzionale `<AppConfig>` e la descrizione 'Descrizione del token app.config' servono a specificare lo scopo del token.

!!! example
    ```ts
    //app.component.ts
    const MY_APP_CONFIG_VARIABLE: AppConfig = {
        title: 'Ciao',
    };

    providers: [{ provide: APP_CONFIG, useValue: MY_APP_CONFIG_VARIABLE }]
    ```  
Viene registrato il provider della dipendenza nel componente usando l’oggetto `InjectionToken` `APP_CONFIG`.

!!! example
    ```ts
    // app.component.ts
    const MY_APP_CONFIG_VARIABLE: AppConfig = {
        title: 'Ciao',
    };

    @Component({
        selector: 'app-root',
        template: `...`,
        providers: [{ provide: APP_CONFIG, useValue: Y_APP_CONFIG_VARIABLE }]
    })
    export class AppComponent {
        apiUrl: string;

        constructor() {
            const config = inject(APP_CONFIG);
            this.apiUrl = config.apiUrl;
        }
    }

    ``` 
Infine la configurazione del token viene inietta nel costruttore tramite la funzione `Inject`.

## Injector

In Angular, l’Injector è il meccanismo centrale che gestisce la creazione e risoluzione delle dipendenze, mantenendo una mappa di provider (token). Normalmente non lo si usa direttamente, perché Angular gestisce automaticamente la Dependency Injection tramite i costruttori. Tuttavia, è possibile fare iniezione manuale usando l’Injector, ad esempio in contesti dinamici o avanzati.

Nel seguente esempio viene illustrata un iniezione manuale con `Inject`.
!!! example
    ```ts
    @Component({
    selector: 'app-example',
    template: `...`,
    providers: [LoggerService]
    })
        export class ExampleComponent {
        constructor(private injector: Injector) {
            const logger = this.injector.get(LoggerService);
        }
    }
    ```
Tramite il metodo `get()` recupera l'istanza del servizio da iniettare.