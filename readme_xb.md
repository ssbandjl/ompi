# create ah failed

[gdr116][[31258,1],4][../../../../../../opal/mca/btl/ofi/btl_ofi_component.c:243:mca_btl_ofi_exit] BTL OFI will now abort.
[gdr116][[31258,1],7][../../../../../../opal/mca/btl/ofi/btl_ofi_component.c:243:mca_btl_ofi_exit] BTL OFI will now abort.
[gdr116][[31258,1],6][../../../../../../opal/mca/btl/ofi/btl_ofi_component.c:243:mca_btl_ofi_exit] BTL OFI will now abort.
[gdr116][[31258,1],5][../../../../../../opal/mca/btl/ofi/btl_ofi_component.c:243:mca_btl_ofi_exit] BTL OFI will now abort.

void mca_btl_ofi_exit(void)
{
    BTL_ERROR(("BTL OFI will now abort."));
    exit(1);
}

mpirun \
  --mca btl ofi,self \
  --mca btl_ofi_verbose 100 \
  --mca btl_base_verbose 100 \
  -np 8 ./your_program

  

create ah stack:
[gdr114] tid:14289, ucreate_cq(), /root/project/rdma/dpu_user_rdma/providers/xtrdma/ucq.c:220, dev_name:xtrdma_0, need_cqe:768, actrue_cqe:1023
[gdr114:14289] select: init of component ofi returned success
[gdr114:14290] mca: bml: Using self btl for send to [[25393,1],1] on node gdr114
[gdr114:14289] mca: bml: Using self btl for send to [[25393,1],0] on node gdr114
[gdr116:10384] mca: bml: Using self btl for send to [[25393,1],3] on node gdr116
[gdr116][[25393,1],3][../../../../../../opal/mca/btl/ofi/btl_ofi_component.c:243:mca_btl_ofi_exit] BTL OFI will now abort.
[gdr114] tid:14290, xtrdma_ucreate_ah(), /root/project/rdma/dpu_user_rdma/providers/xtrdma/uverbs.c:133, VERBS: ibv_cmd_create_ah err 101 attr.global:1 sgid_idx:0
[gdr116:10383] mca: bml: Using self btl for send to [[25393,1],2] on node gdr116
RDMA Stack trace (depth: 14):
/root/project/rdma/dpu_user_rdma/build/lib/libxtrdma-rdmav34.so(+0x38b1)[0x73cb1be1e8b1]
/root/project/rdma/dpu_user_rdma/build/lib/libxtrdma-rdmav34.so(+0x3a95)[0x73cb1be1ea95]
/root/project/rdma/dpu_user_rdma/build/lib/libibverbs.so.1(ibv_create_ah+0x37)[0x73cb20c4d36b]
/lib/x86_64-linux-gnu/libfabric.so.1(+0x807be)[0x73cb1baef7be]
/lib/x86_64-linux-gnu/libfabric.so.1(+0xbdee4)[0x73cb1bb2cee4]
/usr/lib/x86_64-linux-gnu/openmpi/lib/openmpi3/mca_btl_ofi.so(+0x41b6)[0x73cb200141b6]
/usr/lib/x86_64-linux-gnu/openmpi/lib/openmpi3/mca_bml_r2.so(+0x3aae)[0x73cb20d02aae]
/usr/lib/x86_64-linux-gnu/openmpi/lib/openmpi3/mca_pml_ob1.so(mca_pml_ob1_add_procs+0xdc)[0x73cb20c6ad1c]
/lib/x86_64-linux-gnu/libmpi.so.40(ompi_mpi_init+0x87c)[0x73cb38bba51c]
/lib/x86_64-linux-gnu/libmpi.so.40(MPI_Init+0x72)[0x73cb38b50d82]
/root/project/ai/nccl-tests/build/all_reduce_perf(+0x4328)[0x5f7a65681328]
/lib/x86_64-linux-gnu/libc.so.6(+0x2a1ca)[0x73cb3642a1ca]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0x8b)[0x73cb3642a28b]
/root/project/ai/nccl-tests/build/all_reduce_perf(+0x6bd5)[0x5f7a65683bd5]


root@gdr114:~/project/mpi/ompi# addr2line -e /usr/lib/x86_64-linux-gnu/openmpi/lib/openmpi3/mca_btl_ofi.so 0x41b6 -f -C
mca_btl_ofi_add_procs
/build/openmpi-fMmw3z/openmpi-4.1.6/debian/build-gfortran/opal/mca/btl/ofi/../../../../../../opal/mca/btl/ofi/btl_ofi_module.c:91

static int mca_btl_ofi_add_procs (mca_btl_base_module_t *btl,
                                  size_t nprocs, opal_proc_t **opal_procs,
                                  mca_btl_base_endpoint_t **peers,
                                  opal_bitmap_t *reachable)
{
    int rc;
    int count;
    char *ep_name = NULL;
    size_t namelen = mca_btl_ofi_component.namelen;

    opal_proc_t *proc;
    mca_btl_base_endpoint_t *ep;

    mca_btl_ofi_module_t *ofi_btl = (mca_btl_ofi_module_t *) btl;

    for (size_t i = 0 ; i < nprocs ; ++i) {

        proc = opal_procs[i];

        /* See if we already have an endpoint for this proc. */
        rc = opal_hash_table_get_value_uint64 (&ofi_btl->id_to_endpoint, (intptr_t) proc, (void **) &ep);

        if (OPAL_SUCCESS == rc) {
            BTL_VERBOSE(("returning existing endpoint for proc %s", OPAL_NAME_PRINT(proc->proc_name)));
            peers[i] = ep;

        } else {
            /* We don't have this endpoint yet, create one */
            peers[i] = mca_btl_ofi_endpoint_create (proc, ofi_btl->ofi_endpoint);
            BTL_VERBOSE(("creating peer %p", (void*) peers[i]));

            if (OPAL_UNLIKELY(NULL == peers[i])) {
                return OPAL_ERR_OUT_OF_RESOURCE;
            }

            /* Add this endpoint to the lookup table */
            (void) opal_hash_table_set_value_uint64 (&ofi_btl->id_to_endpoint, (intptr_t) proc, (void**) &ep);
        }

        OPAL_MODEX_RECV(rc, &mca_btl_ofi_component.super.btl_version,
                        &peers[i]->ep_proc->proc_name, (void **)&ep_name, &namelen);
        if (OPAL_SUCCESS != rc) {
            BTL_ERROR(("error receiving modex"));
            MCA_BTL_OFI_ABORT();
        }

        /* get peer fi_addr  获取对端地址并插入向量表 */
        count = fi_av_insert(ofi_btl->av,      /* Address vector to insert */
                             ep_name,          /* peer name */
                             1,                /* amount to insert */
                             &peers[i]->peer_addr, /* return peer address here */
                             0,                /* flags */
                             NULL);            /* context */

        /* if succeed, add this proc and mark reachable */
        if (count == 1) { /* we inserted 1 address. */
            opal_list_append (&ofi_btl->endpoints, &peers[i]->super);
            opal_bitmap_set_bit(reachable, i);
        } else {
            BTL_VERBOSE(("fi_av_insert failed with rc = %d", count));
            MCA_BTL_OFI_ABORT();
        }
    }

    return OPAL_SUCCESS;
}



# main
root@gdr114:~/project/mpi/ompi# find . -name orterun
./orte/tools/orterun
./orte/tools/orterun/.libs/orterun
./orte/tools/orterun/orterun
root@gdr114:~/project/mpi/ompi# 

