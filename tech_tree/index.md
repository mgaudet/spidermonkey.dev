---
layout: default
---

<style>
.tree {
    width: 140%;
    text-align: center;
    font-size: 300%; /* This only applies to the placeholder text while render
                        happens */
}

.text {
    width: 60%;
    margin-left: 10%;
}

/* Diagram consumed CSS classes */
.clickable {
    font-style: italic;
    font-size: 98%; /* Adjust for larger space required by italics */
}

.done > * {
    fill: #ceebcf !important;
}

.GC > * {
    fill: #faf3c2 !important;
}

.JIT > * {
    fill: #ffca61 !important;
}

emph {
    font-style: italic;
}

#updated {
    margin-right: 10%;
    text-align: right;
}

</style>

# SpiderMonkey Tech Tree

**Last Updated:** {{site.time | date_to_string }}

## The Tech Tree

<div id="tree" class="tree">Generating SVG</div>
[(Take a peek at the diagram source)](./diagram.mmd)

**What is this?** The above diagram is a thinking tool created by the SpiderMonkey Team to plan out a series of possible futures. **These are not plans**, so much as they are potential routes the project could take.

# Details {#details}

## Shape Indexes {#shapeIndexes}

What if Shapes weren't represented in the object header as a pointer, but instead represented in the object header as table index. This by itself wouldn't gain us much, but would unlock...

## Tagged Shape Indexes {#taggedShapeIndexes}

Since shapes are immutable, we could tag certain information into their Indexes; potentially quite a bit if we were to decide to use a limited size of shape index.

For example, Frozen could be inlined into the tag, removing a dereference

## Universal Relazification {#universalRelazification}

This would be the ability to relazify _any_ script. Currently we can only relazify leaf scripts.

## Regenerate Bytecode For Correctness {#tossBytecode}

The ability to relazifiy anything would allow us to start optimizing _bytecode_, (with the large caveat that we'd need to handle caseswhere optimized functions are on the stack and who knows what that looks like.)

## Immutable Object Detection at Parse Time {#immutableFlag}

In some circumstances we are able to tell that an object is immutable at parse time. If the parser could indicate this we might be able to produce faster code. See [Bug 183095 for some previous investigation.](https://bugzilla.mozilla.org/show_bug.cgi?id=1830195)

## Reusable Inline Caches {#ric}

A [paper published in 2019](https://dl.acm.org/doi/10.1145/3314221.3314587) used a data-driven approach to cache and pre-fill inline cache chains on applications. We could potentially do something similar once we have a disk-cache to store data to.

## Embed Generated Code in Binary {#inBinaryCode}

If we could generate code using our MacroAssembler we could then subsequently also use this to replace inline assembly where we currently use it.

See [this bug](https://bugzilla.mozilla.org/show_bug.cgi?id=1751204).

## SpiderMonkey Relocations {#smRelocations}

In order to be able to generate artifacts to include in our own binaries we either need to generate Position/Context/Runtime independent code, or we need to support our own form of [Relocations](<https://en.wikipedia.org/wiki/Relocation_(computing)>).

## Improved Bytecode {#improvedBytecode}

There is potentially room to improve the performance of our interpreter if we were to invest in improving our bytecode. Some techniques to investigate woudl be breaking apart ops with dynamic behaviour where the dynamism can be identified ahead of time (for example `GetAliasedVar` becoming `GetAliasedVar0`, `GetAliasedVar1` etc.)

We also could investigate macro ops, which encapsulate sequences that have complicated semantics.

### References

- [**Efficient Interpretation with Quickening** Stefan Brunthaler DLS 2010](https://dl.acm.org/doi/10.1145/1899661.1869633) and [followup](https://arxiv.org/pdf/2109.02958.pdf).

## Streaming Parsing {#streamingParsing}

Right now the Parser as designed must see the whole source text before it can parse. It may be beneficial for us to support incremental parsing of a text stream, as we could then parse incoming chunks off the network.

<script type="module">
import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@10.2/dist/mermaid.esm.mjs";
let config = {
    // If you have any issues be sure to set log level to at least 3.
    // Lower is more verbose than higher.
    logLevel: 1, // 5 is default.
    flowcharts: {
    useMaxWidth: true,
    htmlLabels: true,
    },
    // Requried for callbacks
    securityLevel: "loose",
};
mermaid.initialize(config);
window.mermaid = mermaid;

// Cache bust for local development of the diagram.
const url = "./diagram.mmd";
const timestamp = new Date().getTime();
const cacheBustingUrl = `${url}?t=${timestamp}`;

fetch(cacheBustingUrl)
    .then((response) => {
    if (!response.ok) {
        throw new Error(
        `Failed to load file (status code: ${response.status})`
        );
    }
    let diagram_source = response.text();
    return diagram_source;
    })
    .then(async (diagram_source) => {
    let element = document.querySelector("#tree");
    const { svg, bindFunctions } = await mermaid.render(
        "renderedTree",
        diagram_source
    );
    element.innerHTML = svg;
    if (bindFunctions) {
        bindFunctions(element);
    }
    });

// Note: Because of how Mermaid works, callbacks need to be referenced relative to
// window.
window.callbacks = {
    // Callbacks are invoked with the nodeId as the parameter.
    exampleCallback: function (x) {
    alert("called Callback for " + x);
    },
};
</script>