<template>
	<div>
		<h1 class="text-center mt-2 mb-4">Calcs</h1>
		<div>
			<div v-for="calc in calcsData" class="mt-3" :key="calc.slug">
				<label class="form-label h2" :for="calc.slug">Calc {{ calc.slug }}</label>
				<input
					class="form-control"
					type="text"
					:id="calc.slug"
					:value="calc.calc"
					@input="event => {calc.calc = (event.target as HTMLInputElement)!.value; activeData!.find(active => active.slug === calc.slug)!.input = calc.calc}" />
				<h5 class="mt-1">
					Output: <span>{{ calc.output }}</span>
				</h5>
				<hr />
			</div>
		</div>

		<h1 class="text-center mt-4 mb-4">Inputs</h1>

		<form @submit.prevent="submitRecords" class="mt-4">
			<div id="inputContainer" class="row mb-4">
				<div
					v-for="active in activeData"
					:key="active.slug"
					class="mb-4"
					:class="{
					'col-6': calcsData!.find((calc) => calc.slug === active.slug),
					'col-4': !calcsData!.find((calc) => calc.slug === active.slug),
					}">
					<label :for="active.slug" class="form-label">{{ active.slug }}</label>
					<input
						type="text"
						:id="active.slug"
						class="form-control"
						placeholder=""
						:disabled="Boolean(calcsData!.find((calc) => calc.slug === active.slug))"
						:aria-describedby="active + 'Id'"
						v-model="active.input" />
					<small :id="active + 'Id'" class="text-muted">Last reward: {{ last?.[active.slug].y }}</small>
				</div>
			</div>
			<div class="d-grid gap-2">
				<button id="submitButton" type="submit" class="btn btn-outline-success">Submit</button>
			</div>
		</form>

		<pre>{{ activeData }}</pre>
		<pre>{{ calcsData }}</pre>
		<pre>{{ last }}</pre>
	</div>
</template>

<script lang="ts" setup>
import { computedAsync } from '@vueuse/core';

const API_URL = 'http://192.168.178.23:4444/api';

const {
	data: activeData,
	status: activeStatus,
	error: activeError,
	refresh: activeRefresh,
} = await useFetch(`${API_URL}/active`, {
	transform: (data: string[]) =>
		data.map((slug) => {
			return { slug, input: '' };
		}),
});

const {
	data: calcsData,
	status: calcsStatus,
	error: calcsError,
	refresh: calcsRefresh,
} = await useFetch<CalcItem[]>(`${API_URL}/calcs`, {
	transform: (data) => data.map((calc) => ({ ...calc, output: eval(calc.calc) as string })),
});

// const last = ref<Record<string, { x: number; y: number }>>({});

async function submitRecords() {
	const data: Record<string, string> = {};
	for (const active of activeData.value!) {
		data[active.slug] = active.input;
	}
	console.log(data);
	const response = await fetch(`${API_URL}/data`, {
		method: 'POST',
		body: JSON.stringify(data),
		headers: {
			'Content-Type': 'application/json',
		},
	});
	console.log(response);
}

interface CalcItem {
	slug: string;
	calc: string;
	output: string;
}

interface ValueChange<T> {
	old: T;
	new: T;
	changed: boolean;
}

interface ItemChanges {
	calc: ValueChange<string>;
	output: ValueChange<string>;
}

interface ChangeResult {
	slug: string;
	changes: ItemChanges;
}

function findChangedSlugs(newJsonStr: string, oldJsonStr: string): ChangeResult[] {
	try {
		// Parse JSON strings into objects with type assertion
		const newData: CalcItem[] = JSON.parse(newJsonStr);
		const oldData: CalcItem[] = JSON.parse(oldJsonStr);

		// Create maps for easier lookup
		const newMap = new Map<string, CalcItem>(newData.map((item) => [item.slug, item]));
		const oldMap = new Map<string, CalcItem>(oldData.map((item) => [item.slug, item]));

		// Store changed slugs
		const changedSlugs: ChangeResult[] = [];

		// Compare each item in new data with old data
		newMap.forEach((newItem, slug) => {
			const oldItem = oldMap.get(slug);

			// If slug exists in both datasets
			if (oldItem) {
				// Check if any values are different
				if (newItem.calc !== oldItem.calc || newItem.output !== oldItem.output) {
					changedSlugs.push({
						slug,
						changes: {
							calc: {
								old: oldItem.calc,
								new: newItem.calc,
								changed: newItem.calc !== oldItem.calc,
							},
							output: {
								old: oldItem.output,
								new: newItem.output,
								changed: newItem.output !== oldItem.output,
							},
						},
					});
				}
			}
		});

		return changedSlugs;
	} catch (error) {
		throw new Error(`Error processing JSON data: ${error instanceof Error ? error.message : 'Unknown error'}`);
	}
}

watch(
	() => JSON.stringify(calcsData.value),
	(newv, oldv) => {
		console.log(newv, oldv);

		for (const calc of calcsData.value!) {
			try {
				calc.output = eval(calc.calc) as string;
			} catch (error) {}
		}

		const changedSlugs = findChangedSlugs(newv, oldv);

		for (const changedSlug of changedSlugs) {
			const { slug, changes } = changedSlug;
			const body: Record<string, string> = {};
			body[slug] = changes.calc.new;

			fetch(`${API_URL}/calcs`, {
				method: 'POST',
				body: JSON.stringify(body),
				headers: {
					'Content-Type': 'application/json',
				},
			});
		}
	},
	{
		flush: 'sync',
	}
);

const last = computedAsync(async () => {
	const output: Record<string, { x: number; y: number }> = {};
	for (const active of activeData.value!) {
		const lastF = await fetch(`${API_URL}/last/${active.slug}`).then((res) => res.json());
		output[active.slug] = lastF;
		const calcData = calcsData.value!.find((calc) => calc.slug === active.slug);
		if (calcData) {
			active.input = calcData.calc;
		}
	}
	return output;
});
</script>

<style scoped></style>
