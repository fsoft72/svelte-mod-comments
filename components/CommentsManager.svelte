<script lang="ts">
    import DataGrid from '$liwe3/components/DataGrid.svelte';
    import Modal from '$liwe3/components/Modal.svelte';
    import Spinner from '$liwe3/components/Spinner.svelte';

    import { comments_admin_list, comments_admin_del } from '$modules/comments/actions';
    import { onMount } from 'svelte';

    import { storePLANETUser } from '$modules/planet/store.svelte';

    import { Trash, ArrowLeft} from 'svelte-hero-icons';

    import type { DataGridAction, DataGridField, DataGridRow } from '$liwe3/components/DataGrid.svelte'
	import type { CommentType } from '$modules/comments/types';
    import type { PaginatorButtons } from '$liwe3/components/Paginator.svelte';
	import Button from '$liwe3/components/Button.svelte';


    type Props = {
        uid?: string;
        module?: string;
    };

    let { uid, module }: Props = $props();

    let isReady:boolean = $state(false);
    let showConfirm:boolean = $state(false);
    let comments:CommentType[] = $state([]);
    let id:string = $state('');

    const authorFullName = ( name:string, row:DataGridRow ) => {
        const user = storePLANETUser.get(name);
        return user ? `${user.full_name}<br><b>${user.nickname}</b>` : '';
    }

    const gridFields: DataGridField[] = [
        { name: 'id', label: 'ID', type: 'text', hidden: true },
        { name: 'id_obj', label: 'ID', type: 'text', hidden: true },
        { name: 'module', label: 'Module', type: 'text', filterable: true },
        { name: 'id_user', label: 'Author', type: 'text', render: authorFullName, filterable: true },
        { name: 'text', label: 'Comment', type: 'text'}, //, render: (value:string) => short_text(value, 20) },
        { name: 'visible', label: 'Visible', type: 'checkbox' },
    ];

    const gridActions:DataGridAction[] = [
        { label: 'Delete', id: 'delete', icon: Trash, mode: "error",  onclick: (row:DataGridRow) => { deleteComment(row); } },
    ];

    const paginatorButtons: PaginatorButtons = {
		first: { mode: 'mode4', size: 'md', variant: 'outline' },
		prev: { mode: 'mode4', size: 'md', variant: 'outline' },
		next: { mode: 'mode4', size: 'md', variant: 'outline' },
		last: { mode: 'mode4', size: 'md', variant: 'outline' }
	};

    const deleteComment = async (row:DataGridRow) => {
        id = row.id;
        showConfirm = true;
    };

    const _deleteComment = async () => {
        const res = await comments_admin_del(id);
        if (res) {
            await refreshComments();
        }
        showConfirm = false;
    };

    const refreshComments = async () => {
        comments = await comments_admin_list();
    };

    onMount ( async () => {
        await refreshComments();
        isReady = true;
    });
</script>
{#if isReady}
    <DataGrid fields={gridFields} data={comments} actions={gridActions} paginatorButtons={paginatorButtons} />
{:else}
    <Spinner />
{/if}
{#if showConfirm}
    <Modal
        title='Delete comment'
        size='sm'
        mode='error'
        closeOnEsc={false}
        closeOnOutsideClick={false}
        oncancel={(cancel:boolean) => { showConfirm = false; }}
        onclose={(close:boolean) => { showConfirm = false; }}
    >
    <div class="liwe3-row confirm">
        <div class="liwe3-col-xs12">
            <p>Are you sure you want to delete this comment? </p>
        </div>
        <div class="liwe3-offset-xs6 liwe3-col-xs3">
            <Button label='No' size="md" mode='primary' icon={ArrowLeft} onclick={() => { showConfirm = false; }}>NO</Button>
        </div>
        <div class="liwe3-col-xs3">
            <Button label='Yes' size="md" mode='error' icon={Trash} onclick={_deleteComment}>YES</Button>
        </div>
    </div>
    </Modal>
{/if}