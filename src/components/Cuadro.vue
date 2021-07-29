<template>
    <div
        class="cuadro" :class="!info.inicial || info.suspenso ? 'cuadro-vacio': 'cuadro-inicial'">
        <span :class="clase">{{ valor }}</span>
    </div>
</template>
<script>


export default {
    props: ['info'], //info tiene el valor de "item" correspondiente a cada cuadro generado dinamicamente con su respectivo valor, fila, columna y vecinos
    computed: {
        valor () {
            if(this.info.inicial){ 
                if(this.info.bandera){ //tiene bandera?
                    return '🚩'
                }else {
                    return ''
                }
            }
            else {
                
                return this.info.valor
            }
        },
        clase () {

            if(this.info.inicial) {
                if(this.info.inicial) {
                    return 'bandera'
                }
                else {
                    return ''
                }
            }
            else if(this.info.valor == '❌'){
                return 'bandera-falsa'
            }
            else {
                return this.info.claseValor //clase de color numero + color 
            }
   
        }
    }
}
</script>

<style>
    .bandera {
        font-size: 12px;
    }

    .bandera-falsa {
        font-size: 15px;
    }
    .cuadro {
           width: 20px;
        height: 20px;
        display: grid;
        justify-items: center;
        align-items: center;
    }

    .cuadro-inicial {
     
        background-color: #bdbdbd;
        border-top-color: #fff;
        border-left-color: #fff;
        border-right-color: #818181;
        border-bottom-color: #818181;
        border-style: solid;
        border-width: 3px;
        cursor: pointer;
    }

    .cuadro-vacio {
        width: 25px;
        height: 25px;
        background-color: #bdbdbd;
        border-right-color: #7b7b7b;
        border-bottom-color: #7b7b7b;
        border-top-width: 0;
        border-left-width: 0;
        border-bottom-width: 1px;
        border-right-width: 1px;
        border-style: solid;
        cursor:default;
    }
</style>

        <!-- @mousedown.right="bandera" 
        @mouseup.left="activar"
        @contextmenu.prevent -->