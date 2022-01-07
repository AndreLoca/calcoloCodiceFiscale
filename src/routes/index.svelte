<script>
	import { writable } from 'svelte/store';
	import listaComuni from '$lib/data/listaComuni.json';
	import { codificaMesi } from '$lib/mesi';
	import transcodifica from '$lib/data/transcodifica.json';

	let storeComuni = writable(listaComuni);
	let storeTranscodifica = writable(transcodifica);

	let comuneNascita = '';
	let nome = '';
	let cognome = '';
	let data = '';
	let sesso = 'm';
	let codiceComune;

	let anno;
	let mese;
	let giorno;

	$: {
		if (data != '') {
			mese = codificaMesi[parseInt(data.split('-')[1]) - 1].lettera;
			anno = data.split('-')[0].substring(2, 4);
			giorno = sesso == 'm' ? data.split('-')[2] : parseInt(data.split('-')[2]) + 40;
		}
	}

	$: if (comuneNascita.charAt(comuneNascita.length - 1) == ')') {
		let comune = $storeComuni.filter((c) => `${c.nome} (${c.provincia})` == comuneNascita)[0];
		codiceComune = comune.codice;
	} else {
		codiceComune = '';
	}

	let filtraComuni = [];

	$: {
		comuneNascita = comuneNascita.toUpperCase();
		if (comuneNascita.length >= 3) {
			filtraComuni = $storeComuni.filter((comune) => comune.nome.startsWith(comuneNascita));
		} else {
			// ...prende tutti i valori di comuni e li mette nella lista filtraComuni
			filtraComuni = [];
		}
	}

	let nomeCF = '';
	$: {
		let strNomVocali = nome.replace(/[^aeiou]/gi, '').replace(/[^a-z]/gi, '');
		let strNomConsonanti = nome.replace(/[aeiou]/gi, '').replace(/[^a-z]/gi, '');
		console.log(strNomConsonanti);
		console.log(strNomVocali);

		if (strNomConsonanti.length <= 3) {
			// se non ci sono abbastanza vocali metto le x
			if (strNomVocali.length < 3) {
				for (let i = strNomVocali.length; i < 3; i++) {
					strNomVocali += 'X';
				}
			}
			for (let i = strNomConsonanti.length; i < 3; i++) {
				strNomConsonanti += strNomVocali.charAt(0);
				strNomVocali = strNomVocali.substring(1);
			}
			nomeCF = strNomConsonanti.toUpperCase();
		} else {
			nomeCF = `${strNomConsonanti.charAt(0)}${strNomConsonanti.charAt(2)}${strNomConsonanti.charAt(
				3
			)}`.toUpperCase();
		}
	}

	let cognomeCF = '';
	$: {
		let strCgnVocali = cognome.replace(/[^aeiou]/gi, '').replace(/[^a-z]/gi, '');
		let strCgnConsonanti = cognome.replace(/[aeiou]/gi, '').replace(/[^a-z]/gi, '');
		if (strCgnConsonanti.length <= 3) {
			// se non ci sono abbastanza vocali metto le x
			if (strCgnVocali.length < 3) {
				for (let i = strCgnVocali.length; i < 3; i++) {
					strCgnVocali += 'X';
				}
			}
			for (let i = strCgnConsonanti.length; i < 3; i++) {
				strCgnConsonanti += strCgnVocali.charAt(0);
				strCgnVocali = strCgnVocali.substring(1);
			}
			cognomeCF = strCgnConsonanti.toUpperCase();
		} else {
			cognomeCF = strCgnConsonanti.substring(0, 3).toUpperCase();
		}
	}

	let letteraDiControllo;

	$: {
		let somma = 0;
		let merged = cognomeCF + nomeCF + anno + mese + giorno + codiceComune;
		if (merged.length == 15) {
			for (let i = 0; i < merged.length; i++) {
				if ((i + 1) % 2 == 0) {
					let supp = $storeTranscodifica.filter((c) => c.Carattere == merged[i])[0].ValorePari;
					somma += supp;
				} else {
					let supp = $storeTranscodifica.filter((c) => c.Carattere == merged[i])[0].ValoreDispari;
					somma += supp;
				}
			}
			letteraDiControllo = $storeTranscodifica.filter((c) => c.Resto == somma % 26)[0]
				.LetteraDiControllo;
		}
	}
</script>

<div class="flex h-screen">
	<div class="lg:mx-auto my-auto m-5">
		<div class=" bg-gradient-to-r w-fit from-cyan-500 to-green-300 rounded-lg drop-shadow-2xl p-9 border-2 border-sky-900">
			<div class="lg:text-4xl text-2xl text-sky-900 col-span-2 text-center px-6 py-6">
				{cognomeCF}
				{nomeCF}
				{#if anno == undefined}XX{:else}{anno}{/if}
				{#if mese == undefined}XX{:else}{mese}{/if}
				{#if giorno == undefined}XX{:else}{giorno}{/if}
				{#if codiceComune == ''}XXXX{:else}{codiceComune}{/if}
				{#if letteraDiControllo == undefined}X{:else}{letteraDiControllo}{/if}
			</div>
			<div>
				<label for="cognome">Cognome </label><input
					type="text"
					id="cognome"
					placeholder=" es. Rossi"
					bind:value={cognome}
					class="my-2 rounded"
				/>
			</div>
			<div>
				<label for="nome">Nome </label><input
					type="text"
					id="nome"
					placeholder=" es. Luca"
					bind:value={nome}
					class="my-2 rounded"
				/>
			</div>
			<div>
				<div class="">
					<label for="sesso">Sesso:</label>
					<select name="sesso" bind:value={sesso} id="sesso" class="my-2 rounded">
						<option value="m">M</option>
						<option value="f">F</option>
					</select>
				</div>
			</div>
			<div>
				<label for="conasc">Comune di nascita </label>
				<input
					type="text"
					id="conasc"
					placeholder=" es. Milano"
					name="comuneNascita"
					bind:value={comuneNascita}
					list="listaComuni"
					class="my-2 rounded"
				/>
				<datalist id="listaComuni">
					{#each filtraComuni as { nome, provincia }}
						<option value="{nome} ({provincia})">{nome} ({provincia})</option>
					{/each}
				</datalist>
			</div>
			<div>
				<label for="data">Data di nascita </label><input
					type="date"
					id="data"
					placeholder="Data di nascita"
					bind:value={data}
					class="my-2 rounded"
				/>
			</div>
		</div>
	</div>
</div>
