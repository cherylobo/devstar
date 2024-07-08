<script>
  import { writable } from 'svelte/store';
  import html2canvas from 'html2canvas';

  export let nodes = writable([
    { id: 1, text: "Main Topic", x: 500, y: 100, parentId: null, angle: 0 }
  ]);

  let nextId = 2;
  let dragNode = null;
  let newParent = null;
  let title = 'MindMap';

  function addNode(parentId) {
    nodes.update(n => {
      const parentNode = n.find(node => node.id === parentId);
      const newX = parentNode.x + (nextId % 2 === 0 ? -150 : 150);
      const newY = parentNode.y + 150;
      return [
        ...n,
        { id: nextId++, text: `Node ${nextId}`, x: newX, y: newY, parentId: parentId, angle: 0 }
      ];
    });
  }

  function updateText(node, newText) {
    nodes.update(n => {
      const targetNode = n.find(n => n.id === node.id);
      if (targetNode) {
        targetNode.text = newText;
      }
      return [...n];
    });
  }

  function deleteNode(nodeId) {
    nodes.update(n => {
      const remainingNodes = n.filter(node => node.id !== nodeId && node.parentId !== nodeId);
      return remainingNodes.length > 0 ? remainingNodes : n;
    });
  }

  function handleMouseDown(event, node) {
    dragNode = node;
  }

  function handleMouseMove(event) {
    if (dragNode) {
      const container = document.querySelector('.mindmap-container');
      const containerRect = container.getBoundingClientRect();

      let newX = event.clientX - containerRect.left - 75;
      let newY = event.clientY - containerRect.top - 25;

      newX = Math.max(0, Math.min(newX, containerRect.width - 150));
      newY = Math.max(0, Math.min(newY, containerRect.height - 50));

      dragNode.x = newX;
      dragNode.y = newY;
      nodes.update(n => [...n]);
    }
  }

  function handleMouseUp() {
    dragNode = null;
  }

  function setNewParent(node) {
    newParent = node;
  }

  function changeParent(nodeId) {
    if (newParent) {
      nodes.update(n => {
        const targetNode = n.find(node => node.id === nodeId);
        if (targetNode) {
          targetNode.parentId = newParent.id;
        }
        return [...n];
      });
      newParent = null;
    }
  }

  async function downloadMindmap() {
    const mindmapContainer = document.querySelector('.mindmap-container');
    const canvas = await html2canvas(mindmapContainer);
    const pngUrl = canvas.toDataURL('image/png');
    console.log('Current title:', title);
    const downloadLink = document.createElement('a');
    downloadLink.href = pngUrl;
    downloadLink.download = `${title}.png`;
    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
  }

  function updateAngle(node, angle) {
    nodes.update(n => {
      const targetNode = n.find(n => n.id === node.id);
      if (targetNode) {
        targetNode.angle = angle;
      }
      return [...n];
    });
  }
</script>

<style>
  .mindmap-container {
    position: relative;
    width: 100%;
    height: calc(100vh - 64px);
    background-color: #f0f4f8;
    overflow: auto;
    padding: 20px;
    border-radius: 8px;
    border: 2px solid #a0aec0;
  }
  .node {
    position: absolute;
    width: 150px;
    padding: 10px;
    background-color: #ffffff;
    border: 2px solid #007BFF;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s, box-shadow 0.2s;
    cursor: grab;
    text-align: center;
  }
  .node:active {
    cursor: grabbing;
  }
  .node input[type="text"] {
    width: 100%;
    border: none;
    border-bottom: 2px solid #ddd;
    text-align: center;
    outline: none;
  }
  .node input[type="range"] {
    width: 100%;
  }
  .node button {
    margin-top: 10px;
    background-color: #007BFF;
    color: white;
    border: none;
    padding: 7px 9px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  .node button:hover {
    background-color: #0056b3;
  }
  .node .delete-button {
    background-color: #dc3545;
  }
  .node .delete-button:hover {
    background-color: #c82333;
  }
  line {
    stroke: #007BFF;
    stroke-width: 2;
  }
  .arrow {
    marker-end: url(#arrowhead);
  }
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 70px;
    background-color: #ffffff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 0 20px;
    position: sticky;
    top: 0;
    z-index: 10;
    border-radius: 8px;
    border: 2px solid #a0aec0;
  }
  .header .title {
    font-size: 1.2em;
    font-weight: bold;
  }
  .header .actions {
    display: flex;
    gap: 10px;
  }
  .header button {
    background-color: #007BFF;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  .header button:hover {
    background-color: #0056b3;
  }
</style>

<div class="header rounded-lg">
  <div class="title">
    <h1 class="text-blue-800 bg-gray-50 border-2 border-dashed border-gray-500 rounded-lg p-2" contenteditable bind:textContent={title}></h1>
  </div>
  <div class="actions">
    <button class="text-sm font-bold bg-red" on:click={downloadMindmap}>Download</button>
  </div>
</div>

<div
  class="mindmap-container rounded-lg"
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
>
  <svg class="absolute w-full h-full">
    <defs>
      <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="0" refY="3.5" orient="auto">
        <polygon points="0 0, 10 3.5, 0 7" fill="#007BFF" />
      </marker>
    </defs>
    {#each $nodes as node}
      {#if node.parentId !== null}
        {#each $nodes as parent}
          {#if parent.id === node.parentId}
            <line
              class="arrow"
              x1={parent.x + 75}
              y1={parent.y + 25}
              x2={parent.x + 75 + Math.cos(node.angle * Math.PI / 180) * 100}
              y2={parent.y + 25 + Math.sin(node.angle * Math.PI / 180) * 100}
            ></line>
          {/if}
        {/each}
      {/if}
    {/each}
  </svg>
  {#each $nodes as node}
    <div
      class="node"
      style="left: {node.x}px; top: {node.y}px;"
      on:mousedown={(e) => handleMouseDown(e, node)}
    >
      <input 
        type="text" 
        value={node.text} 
        on:input={(e) => updateText(node, e.target.value)}
      />
      <div>
        <button on:click={() => addNode(node.id)}>Add Node</button>
        <button class="delete-button" on:click={() => deleteNode(node.id)}>Delete</button>
        <button on:click={() => setNewParent(node)}>Set as Parent</button>
        {#if newParent && newParent.id !== node.id}
          <button on:click={() => changeParent(node.id)}>Change Parent</button>
        {/if}
        <label>Angle:
          <input 
            type="range" 
            min="0" 
            max="360" 
            value={node.angle} 
            on:input={(e) => updateAngle(node, e.target.value)}
          />
        </label>
      </div>
    </div>
  {/each}
</div>
