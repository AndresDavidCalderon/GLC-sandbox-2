<script setup lang="ts">
import { ref, shallowRef } from 'vue';
const aceptarNumerosTerminal = ref(false);
const aceptarLetrasTerminal = ref(false);
const simbolosTerminales = ref<string[]>([]);
const nuevoSimbolo = ref<string>('');
const reglasDeProduccion = ref<object[]>([])
const nuevaReglaEntrada = ref<string>('');
const nuevaReglaSalida = ref<string>('');
const arbol = shallowRef(null);
const expresion = ref<string>('');

class Token {
	constructor(public tipo: string, public valor: string) {
		this.tipo = tipo;
		this.valor = valor;
	}
}

const agregarSimboloTerminal = (simbolo:string) => {
	if (simbolo != "" && simbolo && !simbolosTerminales.value.includes(simbolo)) {
		simbolosTerminales.value.push(simbolo);
	}
};

const eliminarSimboloTerminal = (index:number) => {
	simbolosTerminales.value = simbolosTerminales.value.toSpliced(index, 1);
};

const agregarReglaProduccion = () => {
	if (nuevaReglaEntrada.value && nuevaReglaSalida.value) {
		reglasDeProduccion.value.push({
			entrada: nuevaReglaEntrada.value,
			salida: nuevaReglaSalida.value
		});
		nuevaReglaEntrada.value = '';
		nuevaReglaSalida.value = '';
	}
};

const sonTokensValidos = (tokens:Array<Token>) => {
	// Verifica que los tokens sean validos, es decir, que todos los terminales sean validos
	let terminales: string = simbolosTerminales.value.join();
	if (aceptarNumerosTerminal.value) {
		terminales += '0123456789';
	}
	if (aceptarLetrasTerminal.value) {
		terminales += 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	}

	const debefaltar = new RegExp(`[^${terminales} ]+$`); // No debe haber caracteres que no sean terminales

	tokens.forEach(token => {
		if (token.tipo === 'terminal' && token.valor.match(debefaltar)!==null) {
			return false
		}
	});
	return true;
};

const analizarExpresion = () => {
	// Primero tokenizacion. La expresion debe volverse un array de tokens, donde cada token esta separado en la expresion por un espacio
	// cada token tiene 2 propiedades: tipo y valor, el tipo puede ser 'terminal' o 'intermedio'.
	const tokens = expresion.value.split(' ').map(token => {
		return { tipo: 'terminal', valor: token };
	})
	if (!sonTokensValidos(tokens)) {
		alert('Los tokens no son validos');
		return;
	}
};

</script>
<template>
	<h1>Describe tu gramatica</h1>
	<h2>Simbolos terminales (F)</h2>
	<p>Aquellos que pueden aparecer en la expresión a partir de la cual construiremos el arbol</p>
	<input type="checkbox" id="aceptar-numeros-terminal" :checked="aceptarNumerosTerminal"
		@change="aceptarNumerosTerminal = $event.target==null ? false : $event.target.checked">
	<label for="aceptar-numeros-terminal">Numeros</label>
	<br>
	<input type="checkbox" id="aceptar-letras-terminal" :checked="aceptarLetrasTerminal"
		@change="aceptarLetrasTerminal = $event.target.checked">
	<label for="aceptar-letras-terminal">Letras</label>
	<br>
	<li v-for="(simbolo, index) in simbolosTerminales" :key="simbolo">
		{{ simbolo }}
		<button @click="eliminarSimboloTerminal(index)">Quitar</button>
	</li>
	<input type="text" placeholder="Añade otro simbolo aquí" :value="nuevoSimbolo"
		@input="nuevoSimbolo = $event.target.value">
	<button @click="() => agregarSimboloTerminal(nuevoSimbolo)">Agregar</button>
	<p>Cada token debe estar separado por un espacio</p>

	<h2>Reglas de produccion</h2>
	<p>Describe que conjunto de simbolos se pueden transformar en otros. Puedes referirte a simbolos terminales con
		[Simbolo aquí]
		o a simbolos intermedios, usa _ para referirte a cualquier elemento final/p>
		<li v-for="(regla, index) in reglasDeProduccion" :key="index">
			{{ regla.entrada }} => {{ regla.salida }}
			<button @click="reglasDeProduccion.splice(index, 1)">Quitar</button>

		</li>
		<input type="text" placeholder="Esto" v-model="nuevaReglaEntrada"> => <input type="text" placeholder="a esto"
			v-model="nuevaReglaSalida" /> <button @click="agregarReglaProduccion"> Agregar</button>

	<h2>Pruebalo</h2>
	<p>Escribe aquí una expresión y mira que arbol genera</p>
	<input type="text" placeholder="Expresión a analizar" v-model="expresion" />
	<button @click="analizarExpresion"> Analizar</button>
</template>