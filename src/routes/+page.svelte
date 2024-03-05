<script lang="ts">
	import { onMount } from "svelte";
	import SemesterTable from "./SemesterTable.svelte";
	import User from "lucide-svelte/icons/user";
	import { Button, buttonVariants } from "$lib/components/ui/button";
	import { Input } from "$lib/components/ui/input";
	import { Textarea } from "$lib/components/ui/textarea";
	import * as Dialog from "$lib/components/ui/dialog";
	import * as Select from "$lib/components/ui/select";
	import { Label } from "$lib/components/ui/label";
	import { clsx } from "clsx";
	import { fade } from "svelte/transition";
	import { toast } from "svelte-sonner";
	import {
		calculateCumulativeGpa,
		type Course,
		parseImportedData,
		formatGpa
	} from "./gpa-calc-utils";
	import LightSwitch from "$lib/components/ui/LightSwitch.svelte";

	let profiles: Record<string, any> = {
		default: { semesters: [[{ name: "", grade: "A", credits: 3 }]] }
	};
	let loaded = false;
	let currentSemester = 0;
	let currentProfile = "default";
	let semesters: Course[][] = [[{ name: "", grade: "A", credits: 3 }]];
	let display = {
		cumulativeGpa: "0.00",
		qualityPoints: "0.00",
		gpaCredits: 0,
		earnedCredits: 0,
		attemptedCredits: 0
	};
	let importDialogOpen = false;
	let addProfileDialogOpen = false;
	let confirmClearDialogOpen = false;
	let confirmDeleteDialogOpen = false;
	let profileDialogOpen = false;

	let profileName = "";
	let importData = "";

	function switchProfile(profileName: string | undefined | any) {
		if (!profileName) profileName = "default";
		currentProfile = profileName;
		semesters = profiles[profileName].semesters;
		currentSemester = 0;
	}

	function checkProfile() {
		if (profileName === "default") {
			toast.error("You cannot use the name 'default'!");
			return false;
		}
		if (profileName.length === 0 || profileName.length > 12) {
			toast.error("Profile name must be between 1 and 12 characters");
			return false;
		}
		if (profiles[profileName]) {
			toast.error("This profile name already exists");
			return false;
		}
		return true;
	}

	function addProfile() {
		const check = checkProfile();
		if (!check) return;
		profiles[profileName] = { semesters: [[{ name: "", grade: "A", credits: 3 }]] };
		toast.success(`${profileName} was created successfully.`);
		switchProfile(profileName);
		addProfileDialogOpen = false;
	}

	function deleteProfile() {
		const toBeDeleted = currentProfile;
		if (toBeDeleted === "default") {
			toast.error("You cannot delete the default profile!");
		} else {
			currentProfile = "default";
			delete profiles[toBeDeleted];
			toast.success(`${toBeDeleted} successfully deleted.`);
		}
		confirmDeleteDialogOpen = false;
	}

	function handleImport() {
		const check = checkProfile();
		if (!check) return;
		semesters = parseImportedData(importData);
		profiles[profileName] = { semesters };
		switchProfile(profileName);
		importDialogOpen = false;
	}

	function handleClear() {
		currentProfile = "default";
		profiles = {
			default: { semesters: [[{ name: "", grade: "A", credits: 3 }]] }
		};
		currentSemester = 0;
		confirmClearDialogOpen = false;
	}

	function addSemester() {
		if (semesters.length === 10) {
			toast.error("You cannot have more than 10 semesters!");
			return;
		}
		semesters.push([{ name: "", grade: "A", credits: 3 }]);
		semesters = semesters;
		currentSemester = semesters.length - 1;
	}

	function deleteSemester() {
		if (semesters.length === 1) {
			toast.error("You cannot delete the only semester!");
			return;
		}
		semesters.splice(currentSemester, 1);
		if (currentSemester !== 0) currentSemester = currentSemester - 1;
		semesters = semesters;
	}

	$: {
		const result = calculateCumulativeGpa(semesters);
		display = {
			cumulativeGpa: formatGpa(result.cumulativeGpa),
			qualityPoints: formatGpa(result.totalQualityPoints),
			gpaCredits: result.totalGpaCredits,
			earnedCredits: result.totalEarnedCredits,
			attemptedCredits: result.totalAttemptedCredits
		};
	}

	$: selectedProfile = { value: currentProfile, label: currentProfile };

	$: semesters = profiles[currentProfile].semesters;

	$: if (loaded) {
		localStorage.setItem("gpa-calculator-profiles", JSON.stringify(profiles));
	}

	onMount(() => {
		const storedProfiles = localStorage.getItem("gpa-calculator-profiles");
		if (storedProfiles) {
			profiles = JSON.parse(storedProfiles);
			const profileNumber = Object.keys(profiles).length;
			if (Object.keys(profiles).length > 0) {
				toast.success(
					`Successfully loaded ${profileNumber} ${
						profileNumber === 1 ? "profile" : "profiles"
					}`
				);
			}
		} else {
			toast.success("Welcome to Gpa Calculator");
		}
		loaded = true;
	});
</script>

<header class="flex items-center justify-between border-b">
	<h1
		class="flex scroll-m-20 justify-between p-2 text-center text-3xl font-semibold tracking-tight transition-colors first:mt-0 sm:pl-4 sm:text-start"
	>
		Gpa Calculator
	</h1>
	<div class="flex items-center justify-center gap-2 px-4">
		<Button
			on:click={() => {
				profileDialogOpen = true;
			}}
			variant="outline"
			class="size-10 p-0"><User /><span class="sr-only">Profile</span></Button
		>
		<LightSwitch />
	</div>
</header>

<main
	class="flex min-h-[80dvh] flex-col items-center bg-gradient-to-t from-zinc-950 via-zinc-900 to-zinc-950 pt-4 sm:pt-16"
>
	<div class="flex flex-col rounded-lg p-2">
		<div class="flex items-center justify-center tabular-nums">
			<div class="grid grid-cols-2 gap-x-2">
				<p>Cumulative GPA</p>
				<p class="text-center">{display.cumulativeGpa}</p>
				<p>GPA Credits</p>
				<p class="text-center">{display.gpaCredits}</p>
				<p>Earned Credits</p>
				<p class="text-center">{display.earnedCredits}</p>
				<p>Attempted Credits</p>
				<p class="text-center">{display.attemptedCredits}</p>
				<p>Total Quality Points</p>
				<p class="text-center">{display.qualityPoints}</p>
			</div>
		</div>
		<div class="flex items-center justify-center gap-2 py-2">
			<Button variant="outline" class="max-w-[8rem] flex-1" on:click={addSemester}
				>Add Semester</Button
			>
			<Button variant="outline" class="max-w-[8rem] flex-1" on:click={deleteSemester}
				>Delete Semester</Button
			>
		</div>
		{#if semesters}
			<div class="flex flex-col items-center justify-center">
				<div class="grid grid-cols-5 items-center justify-center gap-1">
					{#each semesters as _, index}
						<div transition:fade={{ delay: 100 }} class="inline-block px-1 pb-1">
							<Button
								class={clsx("w-10", {
									"bg-emerald-800 hover:bg-emerald-700": currentSemester === index
								})}
								variant="outline"
								on:click={() => {
									currentSemester = index;
								}}>{index + 1}</Button
							>
						</div>
					{/each}
				</div>
				<div>
					{#if semesters[currentSemester]}
						<SemesterTable bind:courses={semesters[currentSemester]} />
					{/if}
				</div>
			</div>
		{/if}
	</div>
</main>

<!-- Main dialog-->
<Dialog.Root
	onOpenChange={() => {
		profileName = "";
	}}
	bind:open={profileDialogOpen}
>
	<Dialog.Content class="rounded-lg">
		<Dialog.Header>
			<Dialog.Title>Profile Settings</Dialog.Title>
		</Dialog.Header>
		<div class="flex flex-col items-center justify-center">
			<Label class="pb-2">Current Profile</Label>
			<Select.Root
				bind:selected={selectedProfile}
				onSelectedChange={(e) => {
					switchProfile(e?.value);
				}}
			>
				<Select.Trigger class="w-[10rem]">
					<Select.Value />
				</Select.Trigger>
				<Select.Content>
					{#each Object.keys(profiles) as profile}
						<Select.Item value={profile}>{profile}</Select.Item>
					{/each}
				</Select.Content>
			</Select.Root>
		</div>
		<div class="flex flex-wrap items-center justify-center gap-2">
			<Button
				variant="outline"
				on:click={() => {
					addProfileDialogOpen = true;
				}}>Add Profile</Button
			>
			<Button
				variant="outline"
				on:click={() => {
					confirmDeleteDialogOpen = true;
				}}>Delete Profile</Button
			>
			<Button
				variant="outline"
				on:click={() => {
					importDialogOpen = true;
				}}>Import</Button
			>
			<Button
				variant="outline"
				on:click={() => {
					confirmClearDialogOpen = true;
				}}>Clear</Button
			>
		</div>
	</Dialog.Content>
</Dialog.Root>

<!-- Add profile dialog -->
<Dialog.Root
	onOpenChange={() => {
		profileName = "";
	}}
	bind:open={addProfileDialogOpen}
>
	<Dialog.Content class="max-w-[16rem] rounded-lg sm:max-w-xs">
		<Dialog.Header>
			<Dialog.Title>Add Profile</Dialog.Title>
			<Dialog.Description>Add a new profile</Dialog.Description>
		</Dialog.Header>
		<div class="grid gap-4">
			<form
				on:submit|preventDefault
				id="add-profile"
				class="grid grid-cols-4 items-center gap-4"
			>
				<Input placeholder="Enter profile name" bind:value={profileName} class="col-span-4" />
			</form>
		</div>
		<Dialog.Footer>
			<Button type="submit" variant="outline" form="add-profile" on:click={addProfile}
				>Add</Button
			>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>

<!-- Delete profile dialog -->
<Dialog.Root bind:open={confirmDeleteDialogOpen}>
	<Dialog.Content class="max-w-[16rem] rounded-lg sm:max-w-xs">
		<Dialog.Header>
			<Dialog.Title>Are you sure?</Dialog.Title>
			<Dialog.Description>This action will delete this profile!</Dialog.Description>
		</Dialog.Header>
		<Dialog.Footer>
			<Button variant="destructive" on:click={deleteProfile}>Delete</Button>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>

<!-- Import dialog -->
<Dialog.Root
	onOpenChange={() => {
		importData = "";
		profileName = "";
	}}
	bind:open={importDialogOpen}
>
	<Dialog.Content class="max-w-[16rem] rounded-lg sm:max-w-[30rem]">
		<Dialog.Header>
			<Dialog.Title>Import</Dialog.Title>
			<Dialog.Description>Add import data</Dialog.Description>
		</Dialog.Header>
		<div class="grid gap-4">
			<div class="grid grid-cols-4 items-center gap-4">
				<Input
					placeholder="Enter profile name"
					bind:value={profileName}
					class="col-span-4 sm:col-span-2"
				/>
				<Textarea
					placeholder="CS201,3,A;MATH201,3,A-|CS204,3,A;CS303,4,B+"
					bind:value={importData}
					class="col-span-4 h-32 resize-none"
				/>
				<!-- -->
			</div>
		</div>
		<Dialog.Footer>
			<Button variant="outline" on:click={handleImport}>Import data</Button>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>

<!-- Clear dialog -->
<Dialog.Root bind:open={confirmClearDialogOpen}>
	<Dialog.Content class="max-w-[16rem] rounded-lg sm:max-w-xs">
		<Dialog.Header>
			<Dialog.Title>Are you sure?</Dialog.Title>
			<Dialog.Description
				>This action will clear all profile data and is irreversible!</Dialog.Description
			>
		</Dialog.Header>
		<Dialog.Footer>
			<Button variant="destructive" on:click={handleClear}>Clear</Button>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>
