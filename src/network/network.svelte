<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import * as tool from '../lib/tool';
  import IconCopy from '../component/iconCopy.svelte';
  import { requestList } from './network.model';
  import Style from './network.less';

  let reqCount = 0;
  const updateReqCount = (list: typeof $requestList) => {
    reqCount = Object.keys(list).length;
  };
  const unsubscribe = requestList.subscribe(updateReqCount);
  updateReqCount($requestList);

  const onTapPreview = (reqId: string) => {
    $requestList[reqId].actived = !$requestList[reqId].actived;
  };

  onMount(() => {
    Style.use();
  });

  onDestroy(() => {
    unsubscribe();
    Style.unuse();
  });

  const prettyStringify = (value: any) => {
    if (tool.isObject(value) || tool.isArray(value)) {
      return tool.safeJSONStringify(value, { maxDepth: 10, keyMaxLen: 10000, pretty: true });
    }
    return value;
  };
</script>

<div class="vc-table">
    
  <dl class="vc-table-row">
    <dd class="vc-table-col vc-table-col-4">Name {#if reqCount > 0}({reqCount}){/if}</dd>
    <dd class="vc-table-col">Method</dd>
    <dd class="vc-table-col">Status</dd>
    <dd class="vc-table-col">Time</dd>
  </dl>

  <div class="vc-plugin-content">
    {#each Object.entries($requestList) as [reqId, req]}
      <div class="vc-group" class:vc-actived="{req.actived}" id="{req.id}">
        <dl class="vc-table-row vc-group-preview" class:vc-table-row-error="{req.status >= 400}" on:click={() => onTapPreview(req.id)}>
          <dd class="vc-table-col vc-table-col-4">{req.name}</dd>
          <dd class="vc-table-col">{req.method}</dd>
          <dd class="vc-table-col">{req.statusText}</dd>
          <dd class="vc-table-col">{req.costTime}</dd>
        </dl>
        <div class="vc-group-detail">
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                General
                <i class="vc-table-row-icon"><IconCopy content={req.url} /></i>
              </dt>
            </dl>
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">URL</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{req.url}</div>
            </div>
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">Method</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{req.method}</div>
            </div>
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">Type</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{req.requestType}</div>
            </div>
          </div>
          {#if (req.header !== null)}
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                Response Headers
                <i class="vc-table-row-icon"><IconCopy content={req.header} /></i>
              </dt>
            </dl>
            {#each Object.entries(req.header) as [key, item]}
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">{key}</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{prettyStringify(item)}</div>
            </div>
            {/each}
          </div>
          {/if}
          {#if (req.requestHeader !== null)}
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                Request Headers
                <i class="vc-table-row-icon"><IconCopy content={req.requestHeader} /></i>
              </dt>
            </dl>
            {#each Object.entries(req.requestHeader) as [key, item]}
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">{key}</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{prettyStringify(item)}</div>
            </div>
            {/each}
          </div>
          {/if}
          {#if (req.getData !== null)}
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                Query String Parameters
                <i class="vc-table-row-icon"><IconCopy content={req.getData} /></i>
              </dt>
            </dl>
            {#each Object.entries(req.getData) as [key, item]}
            <div class="vc-table-row vc-left-border vc-small">
              <div class="vc-table-col vc-table-col-2">{key}</div>
              <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{prettyStringify(item)}</div>
            </div>
            {/each}
          </div>
          {/if}
          {#if (req.postData !== null)}
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                Request Payload
                <i class="vc-table-row-icon"><IconCopy content={req.postData} /></i>
              </dt>
            </dl>
            {#if (typeof req.postData === 'string')}
              <div class="vc-table-row vc-left-border vc-small">
                <pre class="vc-table-col vc-table-col-value vc-max-height-line">{req.postData}</pre>
              </div>
            {:else}
              {#each Object.entries(req.postData) as [key, item]}
              <div class="vc-table-row vc-left-border vc-small">
                <div class="vc-table-col vc-table-col-2">{key}</div>
                <div class="vc-table-col vc-table-col-4 vc-table-col-value vc-max-height-line">{prettyStringify(item)}</div>
              </div>
              {/each}
            {/if}
          </div>
          {/if}
          <div>
            <dl class="vc-table-row vc-left-border">
              <dt class="vc-table-col vc-table-col-title">
                Response
                <i class="vc-table-row-icon"><IconCopy content={req.response} /></i>
              </dt>
            </dl>
            <div class="vc-table-row vc-left-border vc-small">
              <pre class="vc-table-col vc-max-height vc-min-height">{req.response || ''}</pre>
            </div>
          </div>
        </div>
      </div>
    {/each}

  </div>
</div>
