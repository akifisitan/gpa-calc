<script lang="ts">
	import { calculateSemesterGpa, type Course, formatGpa } from "./gpa-calc-utils";
	import Trash2 from "lucide-svelte/icons/trash-2";
	import { Input } from "$lib/components/ui/input";
	import { Button } from "$lib/components/ui/button";
	import { fade, fly } from "svelte/transition";
	import { toast } from "svelte-sonner";

	export let courses: Course[];

	let semesterCredits = 0;
	let semesterQualityPoints = "0.00";
	let semesterGpa = "0.00";

	function calculateTable(courses: Course[]) {
		const result = calculateSemesterGpa(courses);
		semesterCredits = result.semesterCredits;
		semesterQualityPoints = formatGpa(result.semesterQualityPoints);
		semesterGpa = formatGpa(result.semesterGpa);
	}

	function addCourse() {
		if (courses.length >= 10) {
			toast.error("You can only take a maximum of 10 courses per semester");
		} else {
			courses.push({ name: "", grade: "S", credits: 3 });
			courses = courses;
		}
	}

	function deleteCourse(index: number) {
		if (courses.length === 1) {
			toast.error(
				"You cannot delete the single remaining course!\nDelete the semester instead"
			);
		} else {
			courses.splice(index, 1);
			courses = courses;
		}
	}

	$: calculateTable(courses);
</script>

<div
	class="flex flex-col items-center justify-center"
	in:fly={{ x: -100, delay: 200 }}
	out:fly={{ x: 100, duration: 100 }}
>
	<table class="border-separate border-spacing-1">
		<thead>
			<tr>
				<th class="w-[8rem] min-w-[4rem] text-center">Course</th>
				<th class="w-[4rem] min-w-[2rem] text-center">Credits</th>
				<th class="w-[4rem] min-w-[2rem] text-center">Grade</th>
			</tr>
		</thead>
		<tbody>
			{#each courses as course, index}
				<tr transition:fade={{ delay: 100 * index, duration: 300 }}>
					<td
						><Input
							class="text-center"
							bind:value={course.name}
							type="text"
							maxlength={10}
						/></td
					>
					<td
						><Input
							class="text-center"
							bind:value={course.credits}
							type="text"
							maxlength={1}
						/></td
					>
					<td
						><Input
							class="text-center"
							bind:value={course.grade}
							type="text"
							maxlength={2}
						/></td
					>
					<td>
						<Button variant="destructive" on:click={() => deleteCourse(index)}
							><Trash2 /></Button
						>
					</td>
				</tr>
			{/each}
		</tbody>
		<tfoot>
			<tr in:fade={{ delay: 200 * courses.length, duration: 300 }}>
				<td>Total</td>
				<td class="text-center">{semesterCredits}</td>
				<td class="text-center">{semesterGpa}</td>
			</tr>
		</tfoot>
	</table>
	<p class="py-2">Quality Points: {semesterQualityPoints}</p>
	<Button variant="outline" on:click={addCourse}>Add Course</Button>
</div>
