<script lang="ts">
  // type datamap = Record<string, number>[];
  import { onMount } from 'svelte';
  import { BROWSER } from 'esm-env';
  const browser = BROWSER;

  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let url: string;

  export let debug: true;
  export let pause: false;

  export let width = 600;
  export let height = 400;

  type DebugDetails = {
    status: string;
    everything?: boolean;
    event?: Event;
  };
  type DebugMisc = any[];

  const doDebug = (details?: DebugDetails, ...misc: DebugMisc) => {
    if (!debug) return;
    console.debug(details?.status);
    if (details?.everything) {
      console.debug('encoding');
      console.table(encoding);
    }
    if (Array.isArray(misc) && misc.length > 0) {
      console.debug(misc);
    }
  };

  export let htmlID: string = 'deepscatter';

  import type { ZoomAlign, Transform, constant } from './types';
  import { defaultAPICall } from './constants';

  /*----------------------------------------------------------------------------------
   **
   **  NOTE: The following are all defined in the global.d.ts file
   **  of deepscatter. These are all standard types. We will handle the
   **  more complex types below
   **
   **  URL: https://github.com/nomic-ai/deepscatter/blob/main/src/global.d.ts#L270
   **
   **
   **
   **----------------------------------------------------------------------------------*/

  // NOTE: these are note exported to a level we can import
  // import {default_API_call} from 'deepscatter/defaults.ts'
  import type { ZoomCall, Labelcall, Encoding } from 'deepscatter/global.d.ts';

  /** The magnification coefficient for a zooming item */
  export let zoomBalance: number = defaultAPICall.zoomBalance;

  /** The length of time to take for the transition to this state. */
  export let duration: number = defaultAPICall.duration;

  /** The base point size for aes is modified */
  export let pointSize: number = defaultAPICall.pointSize;

  /** The maximum number of points to load */
  export let maxPoints: number = defaultAPICall.maxPoints;

  /** Overall screen saturation target at average point density */
  export let alpha: number = defaultAPICall.alpha;

  /** A function defind as a string that takes implied argument 'datum' */
  export let clickFunction: string | undefined = undefined;

  export let zoomAlign: ZoomAlign = defaultAPICall.zoomAlign;

  export let zoomCall: ZoomCall | undefined = undefined;

  /* NOTE: this type are defined here:
   ** https://github.com/nomic-ai/deepscatter/blob/main/src/global.d.ts#LL245-L267C55
   ** and included in NomicAi/types.ts for convenience
   */
  export let LabelCall: Labelcall = null;

  // NOTE: rather than use APICall.backgroundOptions, we will deconstruct them
  // and then reconstruct them
  export let bgColor: string = defaultAPICall.backgroundOptions.color;
  export let bgOpacity: [number, number] =
    defaultAPICall.backgroundOptions.opacity;
  export let bgSize: [number, number] = defaultAPICall.backgroundOptions.size;
  export let bgMouseover: boolean = defaultAPICall.backgroundOptions.mouseover;
  $: backgroundOptions = {
    color: bgColor,
    opacity: bgOpacity,
    size: bgSize,
    mouseover: bgMouseover,
  };

  /*----------------------------------------------------------------------------------
   **
   **  NOTE: here we deconstrcut the Encoding type and then reconstruct it
   **  URL: https://github.com/nomic-ai/deepscatter/blob/main/src/global.d.ts#L122-L207
   **
   **----------------------------------------------------------------------------------*/

  import type {
    range,
    domain,
    Range,
    Domain,
    Conditional,
    ConditionalChannel,
    LambdaChannel,
    FunctionalChannel,
  } from './types';

  import type { RootChannel, ColorChannel } from 'deepscatter/global.d.ts';

  export let xField: string = 'x';
  export let yField: string = 'y';
  export let cField: string | null = null;

  export let xTransform: Transform | undefined = undefined;
  export let yTransform: Transform | undefined = undefined;
  export let cTransform: Transform | undefined = undefined;

  export let xDomain: domain | undefined = undefined;
  export let yDomain: domain | undefined = undefined;
  export let cDomain: Domain | undefined = undefined;

  export let xRange: range | undefined = undefined;
  export let yRange: range | undefined = undefined;
  export let cRange: Range | undefined = undefined;

  export let xConstant: constant = undefined;
  export let yConstant: constant = undefined;
  export let cConstant: constant = undefined;

  $: xEncoding = {
    field: xField,
    transform: xTransform,
    domain: xDomain,
    range: xRange,
    constant: xConstant,
  } as RootChannel;

  $: yEncoding = {
    field: yField,
    transform: yTransform,
    domain: yDomain,
    range: yRange,
    constant: yConstant,
  } as RootChannel;

  $: cEncoding = {
    field: cField,
    transform: cTransform,
    domain: cDomain,
    range: cRange,
    constant: cConstant,
  } as ColorChannel | null;

  $: encoding = {
    x: xEncoding,
    y: yEncoding,
    color: cEncoding.field === null ? null : cEncoding,
  } as Encoding;

  $: options = {
    alpha,
    duration,
    encoding,

    source_url: url,

    max_points: maxPoints,
    point_size: pointSize,

    zoom_align: zoomAlign,
    zoom_balance: zoomBalance,
    background_options: backgroundOptions,
    background_color: bgColor,

    click_function: clickFunction,
    zoom_call: zoomCall,
    labelcall: LabelCall,
  };

  // NOTE: some events to keep track of
  import type {
    Points,
    Extent,
    Extents,
    DeepScatterReadyEvent,
    DeepScatterExtentEvent,
    DeepScatterSampleEvent,
    DeepScatterFieldsEvent,
    DeepScatterSchemaEvent,
  } from './types';

  // let plot: DeepScatter | null = null
  export let plot: any | null = null;
  export let ready: boolean = false;
  /*
  $: if (browser && Deepscatter !== null) {
    // To handle redraws, we need to destroy the plot and recreate it
    debounceDestroy().then(() => {
      // NOTE: tis gets called a bunch since it is reactive on plot and
      // plot.destory() changes it
      console.warn('Destorying plot and recreating it');
      plot = new Deepscatter(`#${htmlID}`, w, h);
      debouncePlotAPI({ ...options });
    });
  }
  */

  let schema: any = null;
  $: fields = (ready ? plot?._root?.table?.schema?.fields : []) as string[];

  // NOTE: internal props
  $: extents = (ready ? plot?._root?.extent : {}) as Extents;
  $: xExtent = extents?.x || null;
  $: yExtent = extents?.y || null;
  $: zExtent = extents?.z || null;

  $: dispatch('ready', { ready } as DeepScatterReadyEvent);
  $: dispatch('fields', { fields } as DeepScatterFieldsEvent);
  $: dispatch('extents', { extents } as DeepScatterExtentEvent);
  $: dispatch('schema', { schema } as DeepScatterSchemaEvent);

  // NOTE: have to asyncrously import deepscatter to handle SSR
  let Deepscatter = null;
  type DeepScatter = typeof Deepscatter;
  // import Deepscatter from 'deepscatter';
  // import Deepscatter from 'deepscatter?client';
  onMount(async () => {
    const Scatterplot = await import('deepscatter');
    doDebug({ status: 'Deepscatter imported', everything: true }, Scatterplot);
    Deepscatter = Scatterplot.default;
    plot = new Deepscatter(`#${htmlID}`, w, h);
  });

  /**
   * NOTE: these are exposing the deepscatter API in case someone maybe wants to use it's sampling
   * capabilities in "headless" mode (read data and sample, but not use its plotting?)
   */
  export const samplePoints = (n: number) => {
    if (ready && plot) {
      let points = plot?.sample(n);
      dispatch('sample', { points } as DeepScatterSampleEvent);
    }
  };
  export const reZoom = () => {
    if (ready && plot) {
      plot?._zoom.initialize_zoom();
    }
  };

  export const addIdentifierColumn = async (
    field: string,
    datamap: Record<string, number>[],
    reference: string
  ) => {
    doDebug(
      { status: 'addIdentifierColumn', everything: false },
      { field, datamap, reference, plot, ready }
    );
    if (plot && !pause) {
      // await plot?.add_identifier_column(field, datamap, reference)
      plot?.add_identifier_column(field, datamap, reference);
    }
  };

  const destroyWebGLInstances = () => {
    // Get all canvas elements
    const canvasElements = document.getElementsByTagName('canvas');
    if (debug)
      console.debug(
        `Trying to destory WebGL instances of ${canvasElements.length} canvas elements`
      );

    // Loop through each canvas element
    for (let i = 0; i < canvasElements.length; i++) {
      const canvas = canvasElements[i];

      // Get the WebGL context for the canvas
      const context =
        canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

      // Check if a WebGL context exists for the canvas
      if (context) {
        // Call the loseContext method to release WebGL resources
        doDebug({
          status: `Trying to lose WebGL context for canvas ${i}`,
          everything: false,
        });
        context.getExtension('WEBGL_lose_context').loseContext();
      }
    }
  };

  // For trying to autofit the plot to the container
  export let autofit = true;
  // initialize autofitWidth and autofitHeight to be user specified width / height
  // NOTE: these are bound to client width and height of the container, so they will
  // always be distinct from width and height
  let autofitWidth = width;
  let autofitHeight = height;
  $: w = autofit ? autofitWidth : width;
  $: h = autofit ? autofitHeight : height;

  // For trying to debounce the redrawing
  export let delayDestory: number = 250;
  let plotting = false;
  let clearing = false;

  let plottingTimer: ReturnType<typeof setTimeout>;
  let clearingTimer: ReturnType<typeof setTimeout>;

  $: {
    plot
      ?.plotAPI({
        ...options,
      })
      .then(() => (ready = true))
      .then(() => {
        // NOTE: this is were I was adding new identifier columns
      })
      .then(() => {
        // NOTE: since we are in debug mode we will add plot to the window
        window.plot = debug ? plot : null;
      })
      .then(() => (plotting = false))
      .then(async () => {
        schema = await plot?._root.schema();
      });
    // .then(() => plot._zoom.initialize_zoom())
    // .then( () => destroyWebGLInstances())
  }

  // NOTE: this is temporary. We are exposing this for
  // the button panel
  export const doDestory = () => debounceDestroy();
  export const doDraw = () => {
    // Check if we have client and the module
    if (browser && Deepscatter !== null) {
      debounceDestroy().then(() => {
        // NOTE: tis gets called a bunch since it is reactive on plot and
        // plot.destory() changes it
        // plot = new Deepscatter(`#${htmlID}`, w, h);
        debouncePlotAPI({ ...options });
      });
    }
  };

  const destroy = async () => {
    if (plot !== null) {
      // plot.destroy();
      ready = false;
      doDebug(
        { status: 'destroyed plot!', everything: false },
        { ready, plot }
      );
    }
    return Promise.resolve();
  };

  const debounceDestroy = async () => {
    // already clearing, so we will also throttle and stop new calls
    if (clearing) return;
    clearTimeout(clearingTimer);
    clearing = true;
    return new Promise((resolve) => {
      clearingTimer = setTimeout(async () => {
        doDebug({ status: 'debounceDestroy timeout', everything: false });
        destroy().then(() => {
          resolve();
          clearing = false;
          destroyWebGLInstances();
        });
      }, delayDestory);
    });
  };
</script>

<div
  class="flex justify-content-center {$$props.class}"
  bind:clientWidth={autofitWidth}
  bind:clientHeight={autofitHeight}
>
  <div class={$$props.class} style:width="{w}px" style:height="{h}px">
    <div id={htmlID} class="w-full" />
  </div>
</div>
