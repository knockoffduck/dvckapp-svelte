<script>
	import { format, parse } from 'date-fns';
	import io from 'socket.io-client';
	import { onMount } from 'svelte';
	import IoIosArrowBack from 'svelte-icons/io/IoIosArrowBack.svelte';
	import IoIosArrowForward from 'svelte-icons/io/IoIosArrowForward.svelte';
	import { isOverlayOpen } from '../stores/OverlayStore';
	import '../styles/styles.css';
	import { fetchProducts } from '../Tools/Funcs';
	import ProgressBar from './ProgressBar.svelte';

	const url = 'https://dvckapp.xyz/api';
	let current = 0;
	let fetched = false;
	let data;
	let currentShifts;
	let shiftData;
	let len;
	let progress = 0;
	let pressed;
	let socket;

	const clickNext = () => {
		current = current === len - 1 ? 0 : current + 1;
		updateShifts();
	};
	const clickPrev = () => {
		current = current === 0 ? len - 1 : current - 1;
		updateShifts();
	};

	const updateShifts = () => {
		const shiftArray = [];
		shiftData.map((shift) => {
			if (shift.WeekID == data[current].id) {
				shiftArray.push(shift);
			}
		});
		currentShifts = shiftArray;
		if (currentShifts.length == 0) {
			console.log('empty shifts');
		}
	};

	const sendReq = () => {
		console.log(data[current].timetable_url);
		socket.emit('request', `begin, ${data[current].timetable_url}`);
	};

	const addShifts = () => {
		pressed = true;
		socket.emit('request', `addToCalendar,${currentShifts[0].WeekID}`);
	};

	onMount(async () => {
		socket = io('https://dvckapp.xyz', { path: '/ws' });
		try {
			data = await fetchProducts(url);
			shiftData = await fetchProducts('https://dvckapp.xyz/shifts');
			data = data.reverse();
			len = data.length;
			updateShifts();
			fetched = true;
		} catch (err) {
			console.log(error.message);
		}
	});

	$: if (socket) {
		socket.on('progress', (percentage) => {
			progress = parseFloat(percentage);
		});
		socket.on('Completed', async (result) => {
			if (result.includes('Finished')) {
				shiftData = await fetchProducts('https://dvckapp.xyz/shifts');
				updateShifts();
			}
			if (result.includes('Calendar')) {
				isOverlayOpen.set(true);
				pressed = false;
			}
		});
	}
	$: if (progress == 100) {
		progress = 0;
	}
</script>

<div class="bg-[#1A1C21]">
	{#if fetched}
		<div class="grid justify-center h-screen text-white">
			<div class="flex h-[90px] w-full justify-center items-center">
				<h1 class="text-center font-bold text-3xl">Shifts</h1>
			</div>
			<div class="flex flex-col h-[480px] w-full pb-8 gap-4 justify-center place-items-center">
				{#if fetched}
					{#each currentShifts as shift}
						<li class="bg-[url('../public/ShiftBG.svg')] h-[72px] w-[320px] flex text-black">
							<div class="text-center  flex flex-col  justify-center w-[80px]">
								<h5 class="font-bold leading-6 text-2xl">{shift.Day}</h5>
								<h5 class="text-md leading-6">
									{format(parse(shift.Date, 'dd/MM/yy', new Date()), 'dd/MM')}
								</h5>
							</div>
							<div class="flex flex-col w-[150px] justify-center items-center">
								<h5 class="text-sm leading-6">{shift.Start} to {shift.End}</h5>
							</div>
							<div class="flex w-[100px] justify-center items-center">
								<h5 class="font-bold text-xl">{shift.Hours} Hrs</h5>
							</div>
						</li>
					{/each}
				{/if}
			</div>

			<div class="bg-[#25292D] flex flex-col pt-4 w-screen rounded-t-2xl">
				<div class="flex flex-col w-full h-[34px] col-span-2 place-items-center justify-center">
					{#if fetched}
						<h3 class="font-bold text-2xl">
							{format(parse(data[current].date, 'dd MMMM yyyy', new Date()), 'dd MMM')}
						</h3>
					{/if}
				</div>
				<div class="grid grid-cols-2 grid-rows-2 py-5 px-7 gap-6 w-full">
					<button
						on:click={clickNext}
						class="bg-[#393D42] text-white btn flex justify-center items-center shadow-md w-full h-[80px] rounded-lg"
					>
						<div class="w-8 h-8">
							<IoIosArrowBack />
						</div>
					</button>
					<button
						on:click={clickPrev}
						class="bg-[#393D42] text-white btn flex justify-center items-center shadow-md w-full h-[80px] rounded-lg"
					>
						<div class="w-8 h-8">
							<IoIosArrowForward />
						</div>
					</button>

					{#if currentShifts.length == 0}
						<button
							on:click={sendReq}
							class="bg-[#7C7AFF] text-white btn normal-case shadow-md col-span-2 w-full h-[80px] rounded-lg place-items-center justify-center flex"
						>
							{#if !progress}
								<h4 class="text-center text-2xl font-semibold">Get Data</h4>
							{:else}
								<ProgressBar {progress} />
							{/if}
						</button>
					{:else if !pressed}
						<button
							on:click={addShifts}
							class="bg-[#7C7AFF] text-white btn normal-case shadow-md col-span-2 w-full h-[80px] rounded-lg place-items-center justify-center flex"
						>
							<h4 class="text-center text-2xl font-semibold">Add to Calendar</h4>
						</button>
					{:else}
						<button
							class="bg-[#7C7AFF] text-white loading btn normal-case shadow-md col-span-2 w-full h-[80px] rounded-lg place-items-center justify-center flex"
						/>
					{/if}
				</div>
				<div class="grid w-full col-span-2 place-items-center justify-center pt-10 gap-10">
					<h4 class="text-center text-2xl font-semibold">Timetable</h4>
					<div class="grid gap-6 justify-items-center px-7 pb-10">
						{#if fetched}
							<img class="rounded-xl" src={data[current].timetable_url} alt="timetable" />
						{/if}
					</div>
				</div>
			</div>
		</div>
	{:else}
		<div class="w-screen h-screen flex justify-center items-center">
			<button class="btn loading" />
		</div>
	{/if}
</div>
