<template>
  <Page>
    <vue-good-table
      v-if="cols && rows.length > 0"
      :columns="cols"
      :rows="rows"
      styleClass="vgt-table net-table"
      :pagination-options="{
        enabled: true,
        perPage: 30,
        perPageDropdownEnabled: false,
      }"
    >
      <template slot="table-row" slot-scope="props">
        <div v-if="props.column.field == 'asset'" class="cell-content" v-tooltip="props.row.asset">
          <img class="table-asset-icon synth" :src="assetImage(props.row.asset)" alt="asset-icon">
          <span>{{props.formattedRow[props.column.field]}}</span>
        </div>
        <span v-else-if="props.column.field == 'synth'">
          <span v-tooltip="props.row.synth">{{props.formattedRow[props.column.field]}}</span>
        </span>
        <span v-else-if="props.column.field == 'supply'">
          <span>{{props.formattedRow[props.column.field]}}
            <span class="extra-text">
              {{showAsset(props.row.asset)}}
            </span>
          </span>
        </span>
        <span v-else>
          {{props.formattedRow[props.column.field]}}
        </span>
      </template>
    </vue-good-table>
  </Page>
</template>

<script>
import { formatAsset, synthToAsset } from '~/utils';

export default {
  data: function() {
    return {
      cols: [
        {
          label: 'Asset',
          field: 'asset',
          formatFn: formatAsset
        },
        {
          label: 'Synth',
          field: 'synth',
          formatFn: formatAsset,
        },
        {
          label: 'Liability',
          field: 'liability',
          type: 'percentage',
          tdClass: 'mono'
        },
        {
          label: 'Utilisation',
          field: 'utilisation',
          type: 'percentage',
          tdClass: 'mono'
        },
        {
          label: 'Supply',
          field: 'supply',
          type: 'number',
          tdClass: 'mono',
          formatFn: this.numberFormat
        },
      ],
      rows: []
    }
  },
  async asyncData({ $api }) {
    let synthAssets = (await $api.getAssets().catch(e => console.error(e))).data;
    let pools = (await $api.getThorPools().catch(e => console.error(e))).data;

    let synthUtils = []
    for(let asset of synthAssets.supply) {
      let assetName = synthToAsset(asset.denom)
      let pool = pools.find(p => p.asset === assetName)
      if (!pool)
        console.log('Pool hasn\'t been find');
      synthUtils.push({
        asset: assetName,
        synth: asset.denom,
        amount: asset.amount,
        synth_units: pool?.synth_units,
        synth_supply: pool?.synth_supply,
        asset_depth: pool?.balance_asset,
        units: pool?.pool_units,
        supply: asset?.amount,
      })
    }
    return {pools, synthAssets, synthUtils}
  },
  mounted() {
    if (this.synthUtils && this.synthUtils.length > 0) {
      for (let asset of this.synthUtils) {
        if(!asset)
          continue
        this.rows.push({
          asset: asset?.asset,
          synth: asset?.synth,
          liability: +asset?.synth_units/+asset?.units,
          utilisation: +asset?.synth_supply/+asset?.asset_depth,
          supply: +asset?.supply/10**8
        })
      }
    }
  }
}
</script>

<style>
</style>