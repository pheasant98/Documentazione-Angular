# Pipes

In Angular, le pipe sono speciali operatori che consentono di trasformare i dati direttamente nel template, in modo semplice e dichiarativo. Sono molto utili per formattare dati come date, numeri, stringhe, valute, percentuali e altro, senza dover scrivere codice extra nei componenti.<br>
La pipe è una classe con il decoratore @Pipe in cui viene definita una funzione che trasforma valori in input in valori in output da visualizzare in una vista.
Angular offre un set di pipe built-in attreverso il package @angular/common:

| Nome             |  Descrizione                                                                                          |
|------------------|------------------------------------------------------------------------------------------------------|
| AsyncPipe        | Legge il valore da una Promise o da un Observable di RxJS.                                           |
| CurrencyPipe     | Trasforma un numero in una stringa di valuta, formattata secondo le regole locali.                   |
| DatePipe         | Formatta un valore di tipo Date secondo le regole locali.                                            |
| DecimalPipe      | Trasforma un numero in una stringa con punto decimale, formattata secondo le regole locali.          |
| I18nPluralPipe   | Mappa un valore a una stringa che lo pluralizza secondo le regole locali.                            |
| I18nSelectPipe   | Mappa una chiave a un selettore personalizzato che restituisce un valore desiderato.                 |
| JsonPipe         | Trasforma un oggetto in una rappresentazione stringa tramite `JSON.stringify`, utile per il debug.   |
| KeyValuePipe     | Trasforma un oggetto o una mappa in un array di coppie chiave-valore.                                |
| LowerCasePipe    | Trasforma il testo in minuscolo.                                                                     |
| PercentPipe      | Trasforma un numero in una stringa percentuale, formattata secondo le regole locali.                 |
| SlicePipe        | Crea un nuovo array o una nuova stringa contenente una porzione (slice) degli elementi.              |
| TitleCasePipe    | Trasforma il testo in formato titolo (maiuscola all’inizio di ogni parola).                          |
| UpperCasePipe    | Trasforma il testo in maiuscolo.                                                                     |

## Utilizzo 

L'operatore pipe di Angular utilizza il carattere della barra verticale (|) all'interno di un'espressione nel template.
È un operatore binario: l'operando a sinistra rappresenta il valore da trasformare, mentre l'operando a destra indica il nome della pipe e gli eventuali parametri aggiuntivi.

!!! example
    ```html
      <p>Totale: {{ amount | currency }}</p>
    ```

Per specificare un parametro aggiuntivo, basta inserire due punti (:) dopo il nome della pipe, seguiti dal valore del parametro.

!!! example
    ```html
      <p>L'evento si terrà alle {{ scheduledOn | date:'hh:mm' }}.</p>
    ```
Nel caso di più parametri

!!! example
    ```html
      <p>L'evento si terrà alle {{ scheduledOn | date:'hh:mm':'UTC' }}.</p>
    ```

## Pipes Custom

In Angular è possibile definire delle pipes custom.

!!! example
    ```ts
    @Pipe({ name: 'reverse' })
    export class ReversePipe implements PipeTransform {
        transform(value: string): string {
            return value.split('').reverse().join('');
        }
    }

    ```
La pipe ha il nome 'reverse', quindi potrà essere utilizzata nei template con la sintassi {{ valore | reverse }}.<br>
La classe ReversePipe implementa l'interfaccia PipeTransform, il che significa che deve definire un metodo transform. Questo metodo riceve un valore in input, in questo caso una stringa, e restituisce una nuova stringa con i caratteri invertiti. 

Opzionalmente possono essere aggiunto parametri aggiuntivi indicandoli in input nel metodo trasform.

## Pipes Pure

Per ottimizzare le performance, Angular mette a disposizione le pipe pure. Le pipe pure in Angular sono pipe che vengono eseguite solo quando cambia l’input di riferimento, ossia quando cambia effettivamente il valore passato alla pipe (o uno dei suoi argomenti).

Per indicare una pipe come pure basta aggiungere nel decoratore @Pipe, true nel campo pure.

!!! example
    ```ts
    @Pipe({ 
        name: 'reverse', 
        pure: true,
    })
    ```