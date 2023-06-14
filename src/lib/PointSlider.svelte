<script lang="ts">
  let minPoints = 1;
  export let maxPoints = 10000;
  export let numPoints = 500;
  export let step: number | null = null;
</script>

<div class="z--10">
  <label for="num-cell-selector" class="block">
    <span class="text-gray-700">Number of Points</span>
    <span
      class="
            pointer-events-none
            form-input mt-1 block w-full
            rounded-md bg-base-300 border-transparent
            focus:border-base-300 focus:bg-white focus:ring-0"
    >
      {numPoints}
    </span>
  </label>
  <div>
    <input
      id="num-cell-selector"
      type="range"
      min={minPoints}
      max={maxPoints}
      bind:value={numPoints}
      class="range range-primary z--10"
      {step}
    />
    <!-- step="25" -->
    <div class="w-full flex justify-between text-xs px-2">
      <span>|</span>
      <span>|</span>
      <span>|</span>
      <span>|</span>
      <span>|</span>
    </div>
  </div>
</div>
