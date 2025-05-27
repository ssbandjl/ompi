# open mpi note



# ompi_mtl_ofi_get_endpoint
```c
static inline mca_mtl_ofi_endpoint_t *ompi_mtl_ofi_get_endpoint (struct mca_mtl_base_module_t* mtl, ompi_proc_t *ompi_proc)
{
    if (OPAL_UNLIKELY(NULL == ompi_proc->proc_endpoints[OMPI_PROC_ENDPOINT_TAG_MTL])) {
        if (OPAL_UNLIKELY(OMPI_SUCCESS != ompi_mtl_ofi_add_procs(mtl, 1, &ompi_proc))) {
            /* Fatal error. exit() out */
            opal_output(0, "%s:%d: *** The Open MPI OFI MTL is aborting the MPI job (via exit(3)).\n",
                           __FILE__, __LINE__);
            fflush(stderr);
            exit(1);
        }
    }

    return ompi_proc->proc_endpoints[OMPI_PROC_ENDPOINT_TAG_MTL];
}
```


