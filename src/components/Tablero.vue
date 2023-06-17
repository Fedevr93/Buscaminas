<template>
    <div class="tablero">
        <div class="niveles">
            <span>Nivel:</span>
            <span @click="seleccionarNivel(1)" class="nivel" :class="{ 'nivel-seleccionado': nivelActual.nivel == 1 }">1</span>
            <span @click="seleccionarNivel(2)" class="nivel" :class="{ 'nivel-seleccionado': nivelActual.nivel == 2 }">2</span>
            <span @click="seleccionarNivel(3)" class="nivel" :class="{ 'nivel-seleccionado': nivelActual.nivel == 3 }">3</span>
        </div>
        <div class="panel">
            <div class="marcador minas-restantes">
                {{ minasRestantesCifras }}
            </div>
            <div class="cara" @click="iniciarNivel">
                <span>{{cara}}</span>
            </div> <!--🥺-->
            <div class="tiempo">
                {{segundosCifras}}
            </div>
        </div>
        <div class="matriz">
            <cuadro 
                @mousedown.left.native="mouseDownLeft(cuadro)"
                @mousedown.right.native="mouseDownRight(cuadro)"
                @mouseup.left.native="mouseUpLeft(cuadro)"
                @mouseup.right.native="mouseUpRight(cuadro)"
                @mouseenter.native="mouseEnter(cuadro)"
                @mouserleave.native="mouseLeave(cuadro)"
                @contextmenu.prevent.native
                :info="cuadro" v-for="(cuadro, index) in cuadros" :key="index" :style="'grid-row: ' + cuadro.fila + '; grid-column: ' + cuadro.columna + ';'" />
        </div>
    </div>
</template>
<script>
import Cuadro from './Cuadro.vue'

export default {
    components: { Cuadro },
    data () {
        return {
            botonIzquierdo: false,
            botonDerecho: false,
            cara: '😊',
            cuadros: [],
            indices: [],
            totalCuadros: 0,
            colores: ['', 'uno', 'dos', 'tres', 'cuatro', 'cinco', 'seis', 'siete', 'ocho'], 
            nivelPrincipiante: {
                nivel: 1,
                filas: 9, 
                columnas: 9,
                minas: 10
            },
            nivelIntermedio: {
                nivel: 2,
                filas: 16, 
                columnas: 16,
                minas: 40
            },
            nivelExperto: {
                nivel: 3,
                filas: 30, 
                columnas: 30,
                minas: 99
            },
            nivelActual: null,
            minas: [],
            minasRestantes: 0,
            segundos: 0,
            inicio: false,
            timer: null,
            jugando: false,
            cuadrosRestantes: 0
        }
    },
    computed: {
        //Funcion computada para mantener siempre con 3 cifras la cantidad de minas (asi no se deforma el contador)
        minasRestantesCifras () {

            let cifras =  this.minasRestantes.toString()
            if(cifras > 0) {
                if(cifras.length == 1){
                    cifras = '00' + cifras
                }
                else if (cifras.length == 2) {
                        cifras = '0' + cifras
                }
                            
                return cifras
            }
            return '000'
           
        },
        //Funcion computada para mantener siempre con 3 cifras la cuenta del timer (asi no se deforma el contador)
        segundosCifras () {
            let cifras = this.segundos.toString()

            if(cifras.length == 1){
                cifras = '00' + cifras
            }
            else if (cifras.length == 2) {
                cifras = '0' + cifras
            }

            return cifras
        }
    },
    created () {
        this.nivelActual = this.nivelPrincipiante
        this.iniciarNivel()
    },
    methods: {
        
        iniciarNivel() {
            this.botonIzquierdo = false
            this.botonDerecho = false
            this.cara = '😊'
            this.detenerTiempo()
            this.minasRestantes = this.nivelActual.minas //se inicia en la cantidad de minas total
            //Inicia los cuadros
            this.segundos = 0
            this.inicio = false
            console.log("Filas: " + this.nivelActual.filas + "\nColumnas:  " + this.nivelActual.columnas)
            let filas = this.nivelActual.filas
            let columnas =  this.nivelActual.columnas
            let totalCuadros =  filas * columnas

            //Iniciamos los cuadros restantes con el valor total de minas que tiene el nivel seleccionado
            this.cuadrosRestantes = totalCuadros - this.nivelActual.minas

            this.cuadros = []
            this.cuadros = this.crearCuadros(totalCuadros);
            
            console.log(this.cuadros)

//***************************************************************************************************************************************************************************************** */
 

//***************************************************************************************************************************************************************************************** */

        //FACTORIZAR LOGICA QUE EXPARSE MIANS EN EL TABLERO
        
        
        //Funciona: toma el valor que hay en la posicion random del array indices y lo usa como indice del array cuadros para colocar ahi una bomba
            //luego elimina el valor que hay en el indice seleccionao para que no se repita y haya dos bombas en una misma casilla
            //la posicion random se genera entre 0 y el valor correspondiente a la cantidad de indices que quedan para sortear y ubicar una bomba (asi siempre esta eligindo solo en los indices que aun no fueron seleccionados y dentro del rango del array)
            for(let i = 0; i<this.nivelActual.minas; i++){
                let posicion = Math.floor(Math.random() * (this.indices.length - 1)) //genera la posicion random que contendra una bomba
                let indice = this.indices[posicion] 

                this.cuadros[indice].valor = "💣️"; //Coloca la bomba en el cuadro seleccionado de manera random
                this.minas.push(this.cuadros[indice]) 

                this.indices.splice(posicion, 1) //Quita el cuadro de la lista para que no se repita
            }




/******************************************************************************************************************************************************************************************* */
           
         //FACTORIZAR LOGICA QUE SETEA LOS VECINOS CERCANOS Y LA CANTIDAD DE BOMBAS QUE HAY ALREDEDOR DE LA CASILLA

         //Pensar, modificar nombre de "vecinos" por algo mas figurativo de lo que espoiblemente por "bombasAlrededor"
           
           
           //HASTA ACA:
            //Estan esparcidas todas las minas y cada cuadro seleccionado de manera random tiene el valor de "💣️"
            //Inicializa los valores vecinos
            
        
            for (let i = 0; i < totalCuadros; i++){
                
                //seleccionar cuadro 
                let cuadro = this.cuadros[i]
            
                //Ahora vienen una serie de corroboraciones para determinar si el cuadro se encuentra en alguna esquina o borde determinado
                //Para a la hora de setear los vecinos no salirse del rango a una posicion fuera del tablero
                
                
                //Primer If ver esquina superior izquierda, esquina inferior derecha y costado derecho


                //LOGICA 1: CORROBORA POR CADA CUADRO SI ES UNA ESQUINA O BORDE - COMPLETA EL ARRAY VECINOS
                //Borde izquierdo
                if (cuadro.columna == 1){
                    //Esquina superior izquierda
                    if (cuadro.fila == 1){
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas + 1)
                        //Esquina inferior izquierda
                    }else if(cuadro.fila == filas) {
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas + 1)
                        //Costado izquierdo
                    }else {
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas + 1)
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas + 1)
                    }
                    //ver lo que hay en la ultima columna
                }else if(cuadro.columna == columnas) {
                    //ver lo que hay en la esquina superior derecha
                    if (cuadro.fila == 1){
                        cuadro.vecinos.push(i - 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas - 1)
                        //ver esquina inferior derecha
                    }else if (cuadro.fila == filas){
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas - 1)
                        cuadro.vecinos.push(i - 1)
                    }else {
                        //Lo que hay del lado derecho (no esquina)
                        cuadro.vecinos.push(i - 1)
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas - 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas - 1)
                    }
                }
                else {
                    //Borde superior
                    if(cuadro.fila == 1){
                        cuadro.vecinos.push(i - 1)
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas + 1)
                        cuadro.vecinos.push(i + columnas - 1)
                        //Borde inferior
                    }else if(cuadro.fila == filas){
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i - 1)
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas + 1)
                        cuadro.vecinos.push(i - columnas - 1)
                    }else {
                        //No limita con el costado del tablero - todos sus vecinos son cuadros/casillas
                        cuadro.vecinos.push(i + 1)
                        cuadro.vecinos.push(i - 1)
                        cuadro.vecinos.push(i - columnas)
                        cuadro.vecinos.push(i - columnas + 1)
                        cuadro.vecinos.push(i - columnas - 1)
                        cuadro.vecinos.push(i + columnas)
                        cuadro.vecinos.push(i + columnas + 1)
                        cuadro.vecinos.push(i + columnas - 1)
                    }
                }




/******************************************************************************************************************************************************************************************* */




                //LOGICA QUE SETEA LA CANTIDAD DE BOMBAS QUE TIENE ALREDEDOR EL CUADRO


                //Ojo: AL FACTORIZAR TENER EN CUENTA QUE LA LOGICA DE ESTE MODULO ESTA EN BUCLE FOR CON LA LOGICA QUE SETEA LOS VECINOS, ES DECIR
                //CADA VEZ QUE SE SETEAN LOS VECINOS DE UN CUADRO SE SETEA SI ESE CUADRO TIENE BOMBAS ALREDEDOR Y CUANTAS.
                //los vecinos son los numeros necesarios para recueprar los cuadros anteriores

                if(cuadro.valor != '💣️'){
                    //contamos la cantidad de bombas vecinas 
                   let minas = cuadro.vecinos.filter(v => this.cuadros[v].valor == '💣️').length
                  //let minas = 2 // Prueba
                    if( minas > 0 ) {
                        cuadro.valor = minas
                        cuadro.claseValor = 'numero ' + this.colores[minas] //En la posicion 0, no hay clase, en la 1, esta el uno, 2, dos y asi para setear el color
                    }
                
                }

            }

            //PROBAR ESTE COMENTARIO EXISTE ULTIMO COMMIT


            /* CON EL LLAMADO DE ESTA FUNCION INICIA EL JUEGO */
            this.jugando = true

        },

        detenerTiempo () {
            if (this.timer) {
                clearInterval(this.timer)
            }
        },
        crearCuadros(totalCuadros) {
       //FACTORIZAR A UNA FUNCION EXTERNA LOGICA QUECREA EL ARRAY DE CUADROS
         
         //para generar los indices aleatoreos y determinar con el valor de su indice la posicion de las bomba
        console.log(totalCuadros)
          let casillas = []

          for (let i = 0; i <totalCuadros; i++) {

               console.log("VALOR : " + this.nivelActual.columnas)
                let cuadro = {
                    suspenso: false,
                    //Cada componente cuadro individual tendra su propio valor, fila, columna y vecinos registrados
                    inicial: true, //indica si el cuadro esta cubierto o descubierto. esta cubierto asi que true
                    bandera: false,
                    valor: "",
                    fila: Math.floor(i/ this.nivelActual.columnas) + 1,
                    columna: (i % this.nivelActual.columnas) +1,
                    vecinos: [],
                    claseValor: ''
                }

            casillas.push(cuadro)

                //Setea el valor en las minas
            this.indices.push(i) //Colca 1 posicion por cada casillero
                
            }

            return casillas
        },
        agregarSuspenso (cuadro) {
           if (this.jugando) {
               cuadro.suspenso = true

               if (this.botonDerecho) {
                   cuadro.vecinos.forEach(v => {
                       if (!this.cuadros[v].bandera) {
                           this.cuadros[v].suspenso = true
                       }
                   })
               }
               this.cara = '😲'
           }     
        },
        
        quitarSuspesno (cuadro) {
           if (this.jugando) {
            cuadro.suspenso = false

            cuadro.vecinos.forEach(v => {
                this.cuadros[v].suspenso = false
            })
            this.cara = '😊'
           } 
        },
        //Presionar click
        mouseDownLeft(cuadro){
            if (this.jugando) {
                //indico que el boton izquierdo o principal esta siendo clickeado
                this.botonIzquierdo = true

                this.agregarSuspenso(cuadro)
            }
        },
        mouseDownRight(cuadro){
            if (this.jugando) {
                //indiico que el boton derecho esta siendo presionado
                this.botonDerecho = true

                if (!this.botonIzquierdo) {
                    this.cambiarMinasRestantes(cuadro)
                }
                else {
                    this.agregarSuspenso(cuadro)
                }
            }
        },
        //Soltar boton izq (destapar)
        mouseUpLeft(cuadro){
           if (this.jugando) {
               //Indico que el click izquierdo fue levantado
               this.botonIzquierdo = false

               if(cuadro.inicial) {
                   this.activarCuadro(cuadro)
               }
               else if (this.botonDerecho) {
                   this.despejarCuadro(cuadro)
               }
               this.quitarSuspesno(cuadro)
            }
        },
        mouseUpRight(cuadro){
            if (this.jugando) {
                  //  indico que el click dercho fue levantodo
                this.botonDerecho = false
                if (this.botonIzquierdo) {
                    this.despejarCuadro(cuadro)
                }
            }
        },
        mouseEnter(cuadro){
            if (this.jugando && this.botonIzquierdo) {
                this.agregarSuspenso(cuadro)
            }
        },
        mouseLeave(cuadro){
            if (this.jugando && this.cuadro.suspenso) {
                this.quitarSuspesno(cuadro)
            }
        },
        
        seleccionarNivel (nivel) {

            console.log(this.nivelActual.nivel)
            if(this.nivelActual.nivel == nivel) { return }

            if(nivel == 1 ) { this.nivelActual = this.nivelPrincipiante }
            else if (nivel == 2) { this.nivelActual = this.nivelIntermedio }
            else { this.nivelActual = this.nivelExperto }

            this.iniciarNivel()
        },
        activarCuadro (cuadro) {
            //si el cuadro esta tapado y no tiene bandera: destapar
            debugger;
            if (this.jugando && cuadro.inicial && !cuadro.bandera) {
                cuadro.inicial = false

                if(!this.inicio) {
                    this.timer = setInterval(() => {
                        this.segundos++
                    }, 1000)

                    this.inicio = true
                }

                if (cuadro.valor == '💣️' ) {
                    //Explota
                   
                    this.explosion(cuadro)
                }
                else {
                    this.cuadrosRestantes--
                    if(this.cuadrosRestantes <= 0) {
                        this.ganar()
                    }
                    else if (cuadro.valor == '') {
                    //recorrer vecinos y despejarlos
                    debugger;
                    cuadro.vecinos.forEach(v => {
                        this.activarCuadro(this.cuadros[v])
                    })
                }
                }

            }
        },
        despejarCuadro (cuadro) {
            if(!cuadro.inicial && cuadro.valor != ''){
                let banderas = cuadro.vecinos.filter(v => this.cuadros[v].bandera).length 

                

                if (cuadro.valor == banderas){
                    cuadro.vecinos.forEach(v => {
                        this.activarCuadro(this.cuadros[v])
                    })
                }
            }

        },
        explosion(cuadro) {
            this.jugando = false //detiene el juego
            this.detenerTiempo()

            //bucle que recorre el array de minas y las destapa
            this.minas.forEach(mina => {
                if(!mina.bandera) {
                    mina.inicial = false //revela todos las casillas que tienen bombas
                }
            })
            //Recorrer todo el tablero para corroborar el estado de las banderas
            let banderas = this.cuadros.filter(c => c.bandera) //se crea un array de las casillas que contienen banderas

            banderas.forEach(c => {
                //se corrobora que haya una bomba,si todas las banderas por debajo tiene una bomba nunca entra al condicional if y sigue el curso
                // console.log(c.valor)
                console.log(c)
                if (c.valor != '💣️'){
                    c.bandera = false //removemos bandera
                    c.valor = '❌'
                    c.inicial = false
                    console.log(c)//lo guarda pero no lo imprime en el valor
                }
            })
        

            //La bomba clickeada explota y el resto muestra el icono de la bomba
            cuadro.valor = '💥'
            this.cara = '🥺'
        },
        cambiarMinasRestantes (cuadro) {
            //funcin se accede desde el componente cuadro al realizar un click secundario y  colocaro sacar bandera
            if(this.jugando && cuadro.inicial){
                if( this.minasRestantes > 0){
                    //si hay bandera sacar bandera, si no hay bandera poner badnera (true = poner bandera / false = sacar bandera)
                    cuadro.bandera = !cuadro.bandera
                    this.minasRestantes += cuadro.bandera ? -1 : 1 //si se coloca una bandera restar 1 al contador de bandera, si se saca una bandera sumar una al contador de banderas
                } else if (cuadro.bandera) {
                    this.minasRestantes += 1
                    cuadro.bandera = !cuadro.bandera
                }

            }
            
        },
        ganar() {
            this.jugando = false
            this.detenerTiempo()

            this.minas.forEach(mina => {
                mina.bandera = true
            })

            this.minasRestantes = 0
            this.cara = '😎'
        }
    }
}
</script>

<style>

@import url('https://fonts.googleapis.com/css2?family=Roboto+Mono&display=swap');

html { 
    font-family: 'Roboto Mono', monospace;
}


.tablero {
    display: grid;
    justify-content: center;
    background-color: #bdbdbd;
    padding: 10px;
    user-select: none;
}

.niveles { 
    display: grid;
    grid-auto-flow: column;
    grid-gap: 10px;
    font-size: 24px;
    padding: 5px;
    justify-content: start;
    align-items: center;
}

.nivel {
    width: 35px;
    height: 35px;
    color: #5c5c5c;
    text-align: center;
    vertical-align: center;
    cursor: pointer;
}

.nivel-seleccionado {
    color: #fff !important;
    background-color: #5f9cff;
    border-style: solid;
    border-width: 2px;
    border-radius: 50%;
    cursor: default;
}
.panel {
    display:grid;
    grid-auto-flow: column;
    font-size: 30px;
    margin-top: 10px;
    padding: 10px;
    border-top-color: #818181;
    border-left-color: #818181;
    border-bottom-color: #fff;
    border-right-color: #fff;
    border-style: solid;
    border-width: 2px;
}

.marcador {
    background-color: black;
    color: red;
    height: 40px;
    padding:2px;
     border-top-color: #818181;
    border-left-color: #818181;
    border-bottom-color: #fff;
    border-right-color: #fff;
    border-style: solid;
    border-width: 1px;

}

.tiempo {
    justify-self: end;
    background-color: black;
    color: red;
    height: 40px;  
}

.minas-restantes {
    justify-self: start;
}
.cara {
    display: grid;
    justify-content: center;
    align-items: center;
    justify-self: center;
    width: 40px;
    height: 40px;
    font-size: 24px;
    border-top-color: #fff;
    border-left-color: #fff;
    border-bottom-color: #818181;
    border-right-color: #818181;
    border-style: solid;
    border-width: 2px;
    cursor: pointer;
}



.matriz {
    display: grid;
    background-color: #7b7b7b;
    padding: 2px;
    margin-top: 10px;
    border-top-color: #878787;
    border-left-color:#878787;
    border-bottom-color:#fff;
    border-right-color:#fff;
    border-style: solid;
    border-width: 3px;

}
.numero {
    font-size: 20px;
    font-weight: bold;
}
.uno {
    color: blue;
}
.dos {
    color: green;
}
.tres { 
    color: red;
}
.cuatro {
    color: darkblue;
}
.cinco {
    color: brown;
}
.seis {
    color: darkcyan;
}
.siete {
    color: black;
}
.ocho {
    color: black;
}

</style>

//  @onCambiarMinasRestantes="cambiarMinasRestantes" @onActivar="activarCuadro"