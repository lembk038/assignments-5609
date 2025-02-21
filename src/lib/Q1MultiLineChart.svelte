<script lang="ts">
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import type { TMovie } from "../types";

  export let movies: TMovie[] = [];

  let svgElement: SVGSVGElement;

  const margin = { top: 20, right: 30, bottom: 60, left: 60 };
  let width = 700, height = 400;
  let dataByGenre: Record<string, { year: number, count: number }[]> = {};
  let topGenres: string[] = [];

  // Prepare data by aggregating counts by genre and year.
  function prepareData() {
    const counts: Record<string, Record<number, number>> = {};
    movies.forEach(movie => {
      const year = movie.year.getFullYear();
      movie.genres.forEach(genre => {
        if (!counts[genre]) counts[genre] = {};
        counts[genre][year] = (counts[genre][year] || 0) + 1;
      });
    });

    // Compute total counts per genre and determine the top three.
    const totalCounts = Object.entries(counts).map(([genre, yearCounts]) => {
      const total = Object.values(yearCounts).reduce((a, b) => a + b, 0);
      return { genre, total };
    });
    totalCounts.sort((a, b) => b.total - a.total);
    topGenres = totalCounts.slice(0, 3).map(d => d.genre);

    // Build the per-genre time series.
    dataByGenre = {};
    topGenres.forEach(genre => {
      dataByGenre[genre] = [];
      const yearsObj = counts[genre];
      Object.entries(yearsObj).forEach(([year, count]) => {
        dataByGenre[genre].push({ year: +year, count });
      });
      dataByGenre[genre].sort((a, b) => a.year - b.year);
    });
  }

  function drawChart() {
    // Clear the svg.
    d3.select(svgElement).selectAll("*").remove();

    // Prepare data.
    prepareData();

    // Define scales.
    const allYears = topGenres.flatMap(genre => dataByGenre[genre].map(d => d.year));
    const xScale = d3.scaleLinear()
      .domain(d3.extent(allYears) as [number, number])
      .range([margin.left, width - margin.right]);

    const maxCount = d3.max(topGenres.flatMap(genre => dataByGenre[genre].map(d => d.count))) || 1;
    const yScale = d3.scaleLinear()
      .domain([0, maxCount])
      .range([height - margin.bottom, margin.top]);

    const svg = d3.select(svgElement)
      .attr("width", width)
      .attr("height", height);

    // Append x-axis.
    svg.append("g")
      .attr("transform", `translate(0, ${height - margin.bottom})`)
      .call(d3.axisBottom(xScale).tickFormat(d3.format("d")));

    // Append y-axis.
    svg.append("g")
      .attr("transform", `translate(${margin.left}, 0)`)
      .call(d3.axisLeft(yScale));

    // Add x-axis label.
    svg.append("text")
      .attr("text-anchor", "middle")
      .attr("x", width / 2)
      .attr("y", height - margin.bottom + 40)
      .text("Year");

    // Add y-axis label.
    svg.append("text")
      .attr("text-anchor", "middle")
      .attr("transform", "rotate(-90)")
      .attr("x", -height / 2)
      .attr("y", margin.left - 40)
      .text("Movie Count");

    // Line generator.
    const lineGen = d3.line<{ year: number, count: number }>()
      .x(d => xScale(d.year))
      .y(d => yScale(d.count))
      .curve(d3.curveMonotoneX);

    // Draw a line for each top genre.
    topGenres.forEach((genre, index) => {
      svg.append("path")
        .datum(dataByGenre[genre])
        .attr("fill", "none")
        .attr("stroke", d3.schemeCategory10[index])
        .attr("stroke-width", 2)
        .attr("d", lineGen);
    });

    // Add legend for each top genre.
    topGenres.forEach((genre, index) => {
      svg.append("rect")
         .attr("x", width - margin.right - 100)
         .attr("y", margin.top + index * 20)
         .attr("width", 12)
         .attr("height", 12)
         .attr("fill", d3.schemeCategory10[index]);

      svg.append("text")
         .attr("x", width - margin.right - 80)
         .attr("y", margin.top + index * 20 + 10)
         .attr("font-size", "10px")
         .text(genre);
    });
  }

  // Initially attempt to draw when the component mounts.
  onMount(() => {
    if (movies.length > 0) {
      drawChart();
    }
  });

  // Reactive block: redraw the chart whenever movies changes.
  $: if (movies.length > 0) {
    drawChart();
  }
</script>

<svg bind:this={svgElement}></svg>

<style>
  svg {
    border: 1px solid #ccc;
  }
</style>