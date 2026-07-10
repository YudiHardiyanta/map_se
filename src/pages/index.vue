<template>
    <v-container fluid>
        <v-card>
            <v-card-title>Peta Persebaran Titik Sensus Ekonomi Berdasarkan SLS</v-card-title>

            <v-card-text>
                <div style="height: 50px;">
                    <v-btn-toggle v-model="selectedLayer" mandatory color="primary">
                        <v-btn value="osm">
                            Street
                        </v-btn>
                        <v-btn value="satellite">
                            Satellite
                        </v-btn>
                    </v-btn-toggle>
                </div>
                <v-card-text>
                    <v-row>
                        <span>Pilih wilayah sampai minimal SLS</span>
                    </v-row>
                    <v-row>
                        <v-col>
                            <v-select label="Kabupaten/Kota" v-model="selectedKabupaten" :items="kabupatenItems"
                                item-title="title" item-value="value" variant="outlined"
                                @update:model-value="kab_on_change">
                            </v-select>
                        </v-col>
                        <v-col>
                            <v-select label="Kecamatan" v-model="selectedKecamatan" :items="kecamatanItems"
                                item-title="title" item-value="value" variant="outlined"
                                @update:model-value="kec_on_change">
                            </v-select>
                        </v-col>
                        <v-col>
                            <v-select label="Desa/Kelurahan" v-model="selectedDesa" :items="desaItems"
                                item-title="title" item-value="value" variant="outlined"
                                @update:model-value="desa_on_change">
                            </v-select>
                        </v-col>
                        <v-col>
                            <v-select label="SLS" v-model="selectedSLS" :items="slsItems" item-title="title"
                                item-value="value" variant="outlined" @update:model-value="sls_on_change">
                            </v-select>
                        </v-col>
                        <v-col>
                            <v-select label="SUBSLS" v-model="selectedSUBSLS" :items="subslsItems" item-title="title"
                                item-value="value" variant="outlined" @update:model-value="subsls_on_change">
                            </v-select>
                        </v-col>
                    </v-row>



                </v-card-text>


                <div style="height: 600px;">
                    <l-map v-model:zoom="zoom" :center="center" style="height:100%; width:100%;">
                        <!-- OpenStreetMap -->

                        <l-tile-layer :url="tileUrl" />

                        <l-circle-marker v-for="item in points" :key="item.id" :lat-lng="[item.lat, item.long]"
                            :radius="6" color="red" fillColor="red" :fillOpacity="0.8">
                            <l-popup>
                                <v-card width="320">
                                    <v-card-title>
                                        {{ item.nama }}
                                    </v-card-title>

                                    <v-divider />

                                    <v-card-text>
                                        <div><strong>ID</strong> : {{ item.id }}</div>
                                        <div><strong>Code Identity</strong> : {{ item.codeIdentity }}</div>
                                        <div><strong>No Bangunan</strong> : {{ item.no_bangunan }}</div>
                                        <div><strong>Kode Wilayah</strong> : {{ item.kodeWilayah }}</div>
                                        <div><strong>Latitude</strong> : {{ item.lat }}</div>
                                        <div><strong>Longitude</strong> : {{ item.long }}</div>
                                    </v-card-text>
                                </v-card>
                            </l-popup>
                        </l-circle-marker>

                        <l-geo-json :geojson="geoJsonData" :options-style="geoJsonStyle" />

                    </l-map>
                </div>
            </v-card-text>
        </v-card>
    </v-container>
</template>

<script lang="ts" setup>


import { ref, computed } from 'vue'
import {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    LGeoJson,
    LCircleMarker,
} from '@vue-leaflet/vue-leaflet'
import axios from 'axios'

const selectedLayer = ref('osm')

interface SelectItem {
    title: string
    value: string
}

interface Point {
    id: string
    codeIdentity: string
    nama: string
    no_bangunan: string
    kodeWilayah: string
    lat: number
    long: number
}




const kabupatenItems = [
    { title: '01 - Jembrana', value: 5101 },
    { title: '02 - Tabanan', value: 5102 },
    { title: '03 - Badung', value: 5103 },
    { title: '04 - Gianyar', value: 5104 },
    { title: '05 - Klungkung', value: 5105 },
    { title: '06 - Bangli', value: 5106 },
    { title: '07 - Karangasem', value: 5107 },
    { title: '08 - Buleleng', value: 5108 },
    { title: '71 - Denpasar', value: 5171 },
]

const kecamatanItems = ref<SelectItem[]>([])
const desaItems = ref<SelectItem[]>([])
const slsItems = ref<SelectItem[]>([])
const subslsItems = ref<SelectItem[]>([])

const selectedKabupaten = ref<string | null>(null)
const selectedKecamatan = ref<string | null>(null)
const selectedDesa = ref<string | null>(null)
const selectedSLS = ref<string | null>(null)
const selectedSUBSLS = ref<string | null>(null)

const tileUrl = computed(() => {
    switch (selectedLayer.value) {
        case 'satellite':
            return 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}'

        default:
            return 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
    }
})

const zoom = ref(9)
const center = ref([-8.6605, 115.2126])

const geoJsonData = ref(null)

const geoJsonStyle = () => ({
    color: '#E65100',      // garis
    weight: 2,
    opacity: 1,
    fillColor: '#FB8C00',  // isi
    fillOpacity: 0.4
})

const points = ref<Point[]>([])

const change_map = async (value: string) => {
    let map = null;
    if (value.length == 14) {
        map = await axios.get(
            `api/maps/sls_2025_01/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )
        center.value = [map.data.features[0].properties.centroid_y, map.data.features[0].properties.centroid_x]
        geoJsonData.value = map.data
    }
    if (value.length == 16) {
        map = await axios.get(
            `api/maps/subsls_2025_01/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )
        center.value = [map.data.features[0].properties.centroid_y, map.data.features[0].properties.centroid_x]
        geoJsonData.value = map.data
    }

    const titik = await axios.get(
        `api/locations/${value}`,
        {
            headers: {
                Authorization: 'Bearer ipds_ganteng_sekali123!'
            }
        }
    )

    points.value = titik.data
    zoom.value = 20
    //console.log('Map data:', map.data.features[0].geometry.coordinates[0][0])


}

const kab_on_change = async (value: string) => {
    kecamatanItems.value = []
    desaItems.value = []
    slsItems.value = []
    subslsItems.value = []

    selectedKecamatan.value = null
    selectedDesa.value = null
    selectedSLS.value = null
    selectedSUBSLS.value = null

    if (!value) return

    try {
        const response = await axios.get(
            `api/regions/3/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )

        kecamatanItems.value = response.data.map((item: any) => ({
            title: `${item.level_3_fullcode.slice(-3)} - ${item.level_3_name}`,
            value: item.level_3_fullcode,
        }))
    } catch (error) {
        console.error('Error fetching kecamatan data:', error)
    }
}

const kec_on_change = async (value: string) => {
    desaItems.value = []
    slsItems.value = []
    subslsItems.value = []

    selectedDesa.value = null
    selectedSLS.value = null
    selectedSUBSLS.value = null

    if (!value) return

    try {
        const response = await axios.get(
            `api/regions/4/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )

        desaItems.value = response.data.map((item: any) => ({
            title: `${item.level_4_fullcode.slice(-3)} - ${item.level_4_name}`,
            value: item.level_4_fullcode,
        }))
    } catch (error) {
        console.error('Error fetching desa data:', error)
    }
}

const desa_on_change = async (value: string) => {
    selectedSLS.value = null
    selectedSUBSLS.value = null
    slsItems.value = []
    subslsItems.value = []

    if (!value) return

    try {
        const response = await axios.get(
            `api/regions/5/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )

        slsItems.value = response.data.map((item: any) => ({
            title: `${item.level_5_fullcode.slice(-4)} - ${item.level_5_name}`,
            value: item.level_5_fullcode,
        }))
    } catch (error) {
        console.error('Error fetching sls data:', error)
    }
}

const sls_on_change = async (value: string) => {
    selectedSUBSLS.value = null
    subslsItems.value = []

    if (!value) return

    try {
        const response = await axios.get(
            `api/regions/6/${value}`,
            {
                headers: {
                    Authorization: 'Bearer ipds_ganteng_sekali123!'
                }
            }
        )

        subslsItems.value = response.data.map((item: any) => ({
            title: `${item.level_6_fullcode.slice(-2)} - ${item.level_6_name}`,
            value: item.level_6_fullcode,
        }))

        await change_map(value);




    } catch (error) {
        console.error('Error fetching subsls data:', error)
    }
}

const subsls_on_change = async (value: string) => {
    if (!value) return

    try {
        await change_map(value);


    } catch (error) {
        console.error('Error fetching subsls data:', error)
    }
}



</script>
