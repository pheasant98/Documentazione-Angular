# Direttive personalizzate

In Angular, le direttive personalizzate sono classi che permettono di modificare il comportamento o l'aspetto degli elementi del DOM a cui sono associate. Si creano attraverso il decoratore `@Directive`.

!!! example
    ```ts
    import { Directive, ElementRef, Renderer2, HostListener } from '@angular/core';

    @Directive({
    selector: '[appEvidenzia]' // uso: <p appEvidenzia>Testo</p>
    })
    export class EvidenziaDirective {
        constructor(private el: ElementRef, private renderer: Renderer2) {}

        @HostListener('mouseenter') onMouseEnter() {
            this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', 'yellow');
        }

        @HostListener('mouseleave') onMouseLeave() {
            this.renderer.removeStyle(this.el.nativeElement, 'backgroundColor');
        }
    }

    ```

Il codice precedentemente mostrato crea una direttiva personalizzata in Angular per modificare l’aspetto di un elemento HTML al passaggio del mouse. In particolare, questa direttiva imposta uno sfondo giallo quando il cursore si trova sopra l’elemento, e lo rimuove quando il cursore esce. `HostListener` consente di catturare eventi DOM sull’elemento host (come mouseenter, click, ecc.). La variabile `el` rappresenta l’elemento DOM su cui è applicata la direttiva. Il servizio `renderer` consente di modificare il DOM in modo compatibile con le varie piattaforme (es. server-side rendering), infatti permette di impostare lo stile.

Per usare la direttiva in un componente, basta aggiungerla come attributo: 
!!! example
    ```html
    <p appEvidenzia>Passa il mouse su questo paragrafo</p>
    ```


