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
/build/openmpi-fMmw3z/openmpi-4.1.6
