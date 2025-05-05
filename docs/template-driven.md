# Template-driven form

Il Template-driven form in Angular è un approccio per costruire e gestire form in cui la maggior parte della logica è scritta direttamente nell’HTML, sfruttando le direttive offerte da Angular (validator degli input).
I form utilizzano il two-way data binding (binding bidirezionale) per aggiornare il modello di dati nel componente quando vengono apportate modifiche nel template, e viceversa.

!!!info 
    Angular supporta due approcci di progettazione per i form interattivi:

    - I form template-driven ti permettono di usare direttive specifiche per i form direttamente nel template Angular (tratteremo solo questi in questa guida).
     - I form reattivi (reactive forms) forniscono un approccio basato sul modello (model-driven) per la costruzione dei form.

    I form template-driven sono un’ottima scelta per form semplici o di piccole dimensioni, mentre i form reattivi sono più scalabili e adatti a form complessi. 

## Costruzione

Per costruire un form con approcio Template-drive occore utilizzare le direttive come `ngModel`, `NgModelGroup`, `ngForm` del modulo `FormsModule`.
Chiaramente tutto il codice del form sara scritto nel template HTML.

| Direttiva       | Dettagli                                                                                                                                                              |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **NgModel**     | Riconcilia le modifiche di valore nell'elemento del form associato con i cambiamenti nel modello dati, permettendo di gestire l’input utente con validazione ed errori. |
| **NgForm**      | Crea un'istanza di `FormGroup` di livello superiore e la associa a un elemento `<form>` per tracciare i valori aggregati del form e lo stato di validazione. Importando `FormsModule`, questa direttiva è attiva di default su tutti i tag `<form>` non serve aggiungere un selettore speciale. |
| **NgModelGroup**| Crea e associa un'istanza di `FormGroup` a un elemento del DOM.|

Dopo aver importato il modulo `FormsModule` nel root module o comunque nel modulo che contiene il component in cui si vuole implementare il form, il form può essere implemntato nel seguente modo:

!!! example
    ```html
    <form #f="ngForm" (ngSubmit)="onSubmit(f)">
        <label>
            Nome:
            <input name="nome" [(ngModel)]="utente.nome" required #nome="ngModel" />
        </label>    
        <div *ngIf="nome.invalid && nome.touched">
            Il nome è obbligatorio.
        </div>
        <label>
            email:
            <input name="email" [(ngModel)]="utente.email" email />
        </label>

        <h3>Indirizzo</h3>
        <div ngModelGroup="indirizzo">
            <label>
                Via:
                <input name="via" [(ngModel)]="utente.indirizzo.via" required />
            </label>
            <label>
                Città:
                <input name="citta" [(ngModel)]="utente.indirizzo.citta" required />
            </label>
            <label>
                CAP:
                <input name="cap" [(ngModel)]="utente.indirizzo.cap" required />
            </label>
        </div>
        <button type="submit" [disabled]="f.invalid">Invia</button>
    </form>
    ```
    Nel ts la funzione `onSubmit()` è cosi definita:
    ```ts
    export class AppComponent {
        utente = {
            nome: '',
            indirizzo: {
            via: '',
            citta: '',
            cap: ''
            }
        };

        onSubmit(form: any) {
            console.log('Dati inviati:', form.value);
        }
    }
    ```
Dove: 

- `#f="ngForm"`: assegna una reference locale al form.
- `[(ngModel)]="utente.nome"`: crea un two-way binding tra l’input e il modello utente.
- `required`, `email`, ecc.: si usano gli attributi HTML per le validazioni.
- `f.invalid`: Angular gestisce automaticamente la validazione e fornisce proprietà come `valid`, `invalid`, `touched`, `pristine`, ecc.
- `ngModelGroup="indirizzo"` crea un gruppo logico per i campi dell’indirizzo. I nomi degli input (es. "via", "citta", ecc.) diventano sotto-proprietà del gruppo indirizzo nel modello del form.

L’oggetto `form.value` avrà questa struttura:

```ts
{
  nome: 'Mario',
  indirizzo: {
    via: 'Via Roma',
    citta: 'Milano',
    cap: '20100'
  }
}
```
