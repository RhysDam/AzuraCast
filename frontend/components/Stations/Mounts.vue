<template>
    <card-page :title="$gettext('Mount Points')">
        <template #info>
            <p class="card-text">
                {{
                    $gettext('Mount points are how listeners connect and listen to your station. Each mount point can be a different audio format or quality. Using mount points, you can set up a high-quality stream for broadband listeners and a mobile stream for phone users.')
                }}
            </p>
        </template>
        <template #actions>
            <add-button
                :text="$gettext('Add Mount Point')"
                @click="doCreate"
            />
        </template>

        <data-table
            id="station_mounts"
            :fields="fields"
            :provider="listItemProvider"
            paginated
        >
            <template #cell(display_name)="row">
                <h5 class="m-0">
                    <a :href="row.item.links.listen">{{ row.item.display_name }}</a>
                </h5>
                <div v-if="row.item.is_default">
                    <span class="badge text-bg-success">
                        {{ $gettext('Default Mount') }}
                    </span>
                </div>
            </template>
            <template #cell(enable_autodj)="row">
                <template v-if="row.item.enable_autodj">
                    {{ $gettext('Enabled') }} -
                    {{ showFormatAndBitrate(row.item.autodj_format, row.item.autodj_bitrate) }}
                </template>
                <template v-else>
                    {{ $gettext('Disabled') }}
                </template>
            </template>
            <template #cell(actions)="row">
                <div class="btn-group btn-group-sm">
                    <button
                        type="button"
                        class="btn btn-primary"
                        @click="doEdit(row.item.links.self)"
                    >
                        {{ $gettext('Edit') }}
                    </button>
                    <button
                        type="button"
                        class="btn btn-danger"
                        @click="doDelete(row.item.links.self)"
                    >
                        {{ $gettext('Delete') }}
                    </button>
                </div>
            </template>
        </data-table>
    </card-page>

    <edit-modal
        ref="$editModal"
        :create-url="listUrl"
        :new-intro-url="newIntroUrl"
        :station-frontend-type="stationFrontendType"
        @relist="() => relist()"
        @needs-restart="() => mayNeedRestart()"
    />
</template>

<script setup lang="ts">
import DataTable, {DataTableField} from "~/components/Common/DataTable.vue";
import EditModal from "~/components/Stations/Mounts/EditModal.vue";
import {useMayNeedRestart} from "~/functions/useMayNeedRestart";
import {useTranslate} from "~/vendor/gettext";
import {useTemplateRef} from "vue";
import showFormatAndBitrate from "~/functions/showFormatAndBitrate";
import useHasEditModal from "~/functions/useHasEditModal";
import useConfirmAndDelete from "~/functions/useConfirmAndDelete";
import CardPage from "~/components/Common/CardPage.vue";
import {getStationApiUrl} from "~/router";
import AddButton from "~/components/Common/AddButton.vue";
import {FrontendAdapters} from "~/entities/ApiInterfaces.ts";
import {useApiItemProvider} from "~/functions/dataTable/useApiItemProvider.ts";
import {QueryKeys, queryKeyWithStation} from "~/entities/Queries.ts";

defineProps<{
    stationFrontendType: FrontendAdapters
}>();

const listUrl = getStationApiUrl('/mounts');
const newIntroUrl = getStationApiUrl('/mounts/intro');

const {$gettext} = useTranslate();

const fields: DataTableField[] = [
    {key: 'display_name', isRowHeader: true, label: $gettext('Name'), sortable: true},
    {key: 'enable_autodj', label: $gettext('AutoDJ'), sortable: true},
    {key: 'actions', label: $gettext('Actions'), sortable: false, class: 'shrink'}
];

const listItemProvider = useApiItemProvider(
    listUrl,
    queryKeyWithStation([
        QueryKeys.StationMounts
    ])
);

const relist = () => {
    void listItemProvider.refresh();
}

const $editModal = useTemplateRef('$editModal');
const {doCreate, doEdit} = useHasEditModal($editModal);

const {mayNeedRestart} = useMayNeedRestart();

const {doDelete} = useConfirmAndDelete(
    $gettext('Delete Mount Point?'),
    () => {
        mayNeedRestart();
        relist();
    }
);
</script>

