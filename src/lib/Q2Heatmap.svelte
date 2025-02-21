<script lang="ts">
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import type { TMovie } from "../types";

  export let movies: TMovie[] = [];
  let svgElement: SVGSVGElement;
  const margin = { top: 50, right: 50, bottom: 50, left: 50 };
  let width = 600, height = 600;
  let genres: string[] = [];
  let matrix: number[][] = []; // Co-occurrence matrix.

  function prepareData() {
    // Collect unique genres.
    const genreSet = new Set<string>();
    movies.forEach(movie => {
      movie.genres.forEach(genre => genreSet.add(genre));
    });
    genres = Array.from(genreSet).sort();

    // Initialize matrix.
    matrix = genres.map(() => genres.map(() => 0));

    // Compute co-occurrence.
    movies.forEach(movie => {
      const g = movie.genres;
      g.forEach((genreA) => {
        g.forEach((genreB) => {
          if (genreA !== genreB) {
            const idxA = genres.indexOf(genreA);
            const idxB = genres.indexOf(genreB);
            matrix[idxA][idxB] += 1;
          }
        });
      });
    });
  }

  function drawHeatmap() {
    // Clear the svg.
    d3.select(svgElement).selectAll("*").remove();

    // Prepare data.
    prepareData();

    const gridSize = Math.floor((width - margin.left - margin.right) / genres.length);
    
    // Flatten matrix for binding.
    const heatmapData = [];
    for (let i = 0; i < genres.length; i++) {
      for (let j = 0; j < genres.length; j++) {
        heatmapData.push({
          genreX: genres[j],
          genreY: genres[i],
          value: matrix[i][j]
        });
      }
    }
    
    const colorScale = d3.scaleSequential(d3.interpolateOrRd)
      .domain([0, d3.max(heatmapData, d => d.value) || 1]);

    const svg = d3.select(svgElement)
      .attr("width", width)
      .attr("height", height);

    // Draw cells.
    svg.selectAll("rect")
      .data(heatmapData)
      .enter()
      .append("rect")
      .attr("x", d => margin.left + genres.indexOf(d.genreX) * gridSize)
      .attr("y", d => margin.top + genres.indexOf(d.genreY) * gridSize)
      .attr("width", gridSize)
      .attr("height", gridSize)
      .attr("fill", d => colorScale(d.value));

    // Add value labels.
    svg.selectAll("text.cell")
      .data(heatmapData)
      .enter()
      .append("text")
      .attr("class", "cell")
      .attr("x", d => margin.left + genres.indexOf(d.genreX) * gridSize + gridSize / 2)
      .attr("y", d => margin.top + genres.indexOf(d.genreY) * gridSize + gridSize / 2)
      .attr("text-anchor", "middle")
      .attr("dy", ".35em")
      .attr("font-size", "10px")
      .text(d => d.value > 0 ? d.value : "");

    // Add x-axis labels rotated vertically.
    svg.selectAll(".xLabel")
      .data(genres)
      .enter()
      .append("text")
      .attr("class", "xLabel")
      .attr("transform", (d, i) => {
          const x = margin.left + i * gridSize + gridSize / 2;
          // Move the x-axis labels further upward (e.g., margin.top - 15)
          const y = margin.top - 15;
          return `translate(${x}, ${y}) rotate(-90)`;
      })
      .attr("text-anchor", "middle")
      .attr("font-size", "10px")
      .text(d => d);

    // Add y-axis labels.
    svg.selectAll(".yLabel")
      .data(genres)
      .enter()
      .append("text")
      .attr("class", "yLabel")
      .attr("x", margin.left - 5)
      .attr("y", (d, i) => margin.top + i * gridSize + gridSize / 2)
      .attr("text-anchor", "end")
      .attr("font-size", "10px")
      .text(d => d);
  }

  onMount(() => {
    if (movies.length > 0) {
      drawHeatmap();
    }
  });

  // Reactive block to redraw heatmap when movies changes.
  $: if (movies.length > 0) {
    drawHeatmap();
  }
</script>

<svg bind:this={svgElement}></svg>

<style>
  svg {
    border: 1px solid #ccc;
  }
</style>