# main entry
```c
ompi/tools/mpirun/main.c
```


# btl
```c
/**
 * BTL module interface functions and attributes.
 */
struct mca_btl_base_module_t {

    /* BTL common attributes */
    mca_btl_base_component_t *btl_component; /**< pointer back to the BTL component structure */
    size_t btl_eager_limit;                  /**< maximum size of first fragment -- eager send */
    size_t btl_rndv_eager_limit; /**< the size of a data sent in a first fragment of rendezvous
                                    protocol */
    size_t btl_max_send_size;    /**< maximum send fragment size supported by the BTL */
    size_t btl_rdma_pipeline_send_length; /**< amount of bytes that should be send by pipeline
                                             protocol */
    size_t btl_rdma_pipeline_frag_size;   /**< maximum rdma fragment size supported by the BTL */
    size_t btl_min_rdma_pipeline_size;    /**< minimum packet size for pipeline protocol  */
    uint32_t btl_exclusivity;             /**< indicates this BTL should be used exclusively */
    uint32_t btl_latency;                /**< relative ranking of latency used to prioritize btls */
    uint32_t btl_bandwidth;              /**< bandwidth (Mbytes/sec) supported by each endpoint */
    uint32_t btl_flags;                  /**< flags (put/get...) */
    uint32_t btl_atomic_flags;           /**< atomic operations supported (add, and, xor, etc) */
    size_t btl_registration_handle_size; /**< size of the BTLs registration handles */

    /* One-sided limitations (0 for no alignment, SIZE_MAX for no limit ) */
    size_t btl_get_limit;     /**< maximum size supported by the btl_get function */
    size_t btl_get_alignment; /**< minimum alignment/size needed by btl_get (power of 2) */
    size_t btl_put_limit;     /**< maximum size supported by the btl_put function */
    size_t btl_put_alignment; /**< minimum alignment/size needed by btl_put (power of 2) */

    /* minimum transaction sizes for which registration is required for local memory */
    size_t btl_get_local_registration_threshold;
    size_t btl_put_local_registration_threshold;

    /* BTL function table */
    mca_btl_base_module_add_procs_fn_t btl_add_procs;
    mca_btl_base_module_del_procs_fn_t btl_del_procs;
    mca_btl_base_module_register_fn_t btl_register;
    mca_btl_base_module_finalize_fn_t btl_finalize;

    mca_btl_base_module_alloc_fn_t btl_alloc;
    mca_btl_base_module_free_fn_t btl_free;
    mca_btl_base_module_prepare_fn_t btl_prepare_src;
    mca_btl_base_module_send_fn_t btl_send;
    mca_btl_base_module_sendi_fn_t btl_sendi;
    mca_btl_base_module_put_fn_t btl_put;
    mca_btl_base_module_get_fn_t btl_get;
    mca_btl_base_module_dump_fn_t btl_dump;

    /* atomic operations */
    mca_btl_base_module_atomic_op64_fn_t btl_atomic_op;
    mca_btl_base_module_atomic_fop64_fn_t btl_atomic_fop;
    mca_btl_base_module_atomic_cswap64_fn_t btl_atomic_cswap;

    /* new memory registration functions */
    mca_btl_base_module_register_mem_fn_t
        btl_register_mem; /**< memory registration function (NULL if not needed) */
    mca_btl_base_module_deregister_mem_fn_t
        btl_deregister_mem; /**< memory deregistration function (NULL if not needed) */

    /** the mpool associated with this btl (optional) */
    mca_mpool_base_module_t *btl_mpool;
    /** register a default error handler */
    mca_btl_base_module_register_error_fn_t btl_register_error;
#if OPAL_CUDA_GDR_SUPPORT
    size_t btl_accelerator_eager_limit; /**< switch from eager to RDMA */
    size_t btl_accelerator_rdma_limit;  /**< switch from RDMA to rndv pipeline */
#endif /* OPAL_CUDA_GDR_SUPPORT */
    size_t btl_accelerator_max_send_size; /**< set if CUDA max send_size is different from host max send
                                      size */

    mca_btl_base_module_flush_fn_t btl_flush; /**< flush all previous operations on an endpoint */


    union {
        struct {
            void *btl_am_data;
        };
        unsigned char padding[256]; /**< padding to future-proof the
                                       btl module */
    };
};
```

# mca_bml_base_put
```c
mca_bml_base_put

```

# MPI_PUT
```c
ompi/mpi/c/put.c
int MPI_Put(const void *origin_addr, int origin_count, MPI_Datatype origin_datatype,
            int target_rank, MPI_Aint target_disp, int target_count,
            MPI_Datatype target_datatype, MPI_Win win)
{
    int rc;

    SPC_RECORD(OMPI_SPC_PUT, 1);

    if (MPI_PARAM_CHECK) {
        rc = OMPI_SUCCESS;

        OMPI_ERR_INIT_FINALIZE(FUNC_NAME);

        if (ompi_win_invalid(win)) {
            return OMPI_ERRHANDLER_NOHANDLE_INVOKE(MPI_ERR_WIN, FUNC_NAME);
        } else if (origin_count < 0 || target_count < 0) {
            rc = MPI_ERR_COUNT;
        } else if (ompi_win_peer_invalid(win, target_rank) &&
                   (MPI_PROC_NULL != target_rank)) {
            rc = MPI_ERR_RANK;
        } else if (NULL == target_datatype ||
                   MPI_DATATYPE_NULL == target_datatype) {
            rc = MPI_ERR_TYPE;
        } else if ( MPI_WIN_FLAVOR_DYNAMIC != win->w_flavor && target_disp < 0 ) {
            rc = MPI_ERR_DISP;
        } else {
            OMPI_CHECK_DATATYPE_FOR_ONE_SIDED(rc, origin_datatype, origin_count);
            if (OMPI_SUCCESS == rc) {
                OMPI_CHECK_DATATYPE_FOR_ONE_SIDED(rc, target_datatype, target_count);
            }
        }
        OMPI_ERRHANDLER_CHECK(rc, win, rc, FUNC_NAME);
    }

    if (MPI_PROC_NULL == target_rank) return MPI_SUCCESS;

    rc = win->w_osc_module->osc_put(origin_addr, origin_count, origin_datatype,
                                    target_rank, target_disp, target_count,
                                    target_datatype, win);
    OMPI_ERRHANDLER_RETURN(rc, win, rc, FUNC_NAME);
}
```

# btl put
```c
static inline int
ompi_osc_rdma_btl_put(ompi_osc_rdma_module_t *module, uint8_t btl_index,
		      struct mca_btl_base_endpoint_t *endpoint,
		      void *local_address, uint64_t remote_address,
		      struct mca_btl_base_registration_handle_t *local_handle,
		      struct mca_btl_base_registration_handle_t *remote_handle,
		      size_t size, int flags, int order,
		      mca_btl_base_rdma_completion_fn_t cbfunc,
		      void *cbcontext, void *cbdata)
{
    if (module->use_accelerated_btl) {
        mca_btl_base_module_t *btl = ompi_osc_rdma_selected_btl(module, btl_index);
        return btl->btl_put(btl, endpoint, local_address, remote_address,
                            local_handle, remote_handle, size, flags, order,
                            cbfunc, cbcontext, cbdata);
    } else {
        mca_btl_base_am_rdma_module_t *am_rdma = ompi_osc_rdma_selected_am_rdma(module, btl_index);
        return am_rdma->am_btl_put(am_rdma, endpoint, local_address, remote_address,
                                   local_handle, remote_handle, size, flags, order,
                                   cbfunc, cbcontext, cbdata);
    }
}
```


# one-side communication(OSC) interface
```c
/* ******************************************************************** */


/**
 * OSC module instance
 *
 * Module interface to the OSC system.  An instance of this module is
 * attached to each window.  The window contains a pointer to the base
 * module instead of a base module itself so that the component is
 * free to create a structure that inherits this one for use as the
 * module structure.
 */
struct ompi_osc_base_module_4_0_0_t {
    ompi_osc_base_module_win_shared_query_fn_t osc_win_shared_query;

    ompi_osc_base_module_win_attach_fn_t osc_win_attach;
    ompi_osc_base_module_win_detach_fn_t osc_win_detach;
    ompi_osc_base_module_free_fn_t osc_free;

    ompi_osc_base_module_put_fn_t osc_put;
    ompi_osc_base_module_get_fn_t osc_get;
    ompi_osc_base_module_accumulate_fn_t osc_accumulate;
    ompi_osc_base_module_compare_and_swap_fn_t osc_compare_and_swap;
    ompi_osc_base_module_fetch_and_op_fn_t osc_fetch_and_op;
    ompi_osc_base_module_get_accumulate_fn_t osc_get_accumulate;

    ompi_osc_base_module_rput_fn_t osc_rput;
    ompi_osc_base_module_rget_fn_t osc_rget;
    ompi_osc_base_module_raccumulate_fn_t osc_raccumulate;
    ompi_osc_base_module_rget_accumulate_fn_t osc_rget_accumulate;

    ompi_osc_base_module_fence_fn_t osc_fence;

    ompi_osc_base_module_start_fn_t osc_start;
    ompi_osc_base_module_complete_fn_t osc_complete;
    ompi_osc_base_module_post_fn_t osc_post;
    ompi_osc_base_module_wait_fn_t osc_wait;
    ompi_osc_base_module_test_fn_t osc_test;

    ompi_osc_base_module_lock_fn_t osc_lock;
    ompi_osc_base_module_unlock_fn_t osc_unlock;
    ompi_osc_base_module_lock_all_fn_t osc_lock_all;
    ompi_osc_base_module_unlock_all_fn_t osc_unlock_all;

    ompi_osc_base_module_sync_fn_t osc_sync;
    ompi_osc_base_module_flush_fn_t osc_flush;
    ompi_osc_base_module_flush_all_fn_t osc_flush_all;
    ompi_osc_base_module_flush_local_fn_t osc_flush_local;
    ompi_osc_base_module_flush_local_all_fn_t osc_flush_local_all;
};
```


# ofi put
```c

int mca_btl_ofi_put(mca_btl_base_module_t *btl, mca_btl_base_endpoint_t *endpoint,
                    void *local_address, uint64_t remote_address,
                    mca_btl_base_registration_handle_t *local_handle,
                    mca_btl_base_registration_handle_t *remote_handle, size_t size, int flags,
                    int order, mca_btl_base_rdma_completion_fn_t cbfunc, void *cbcontext,
                    void *cbdata)
{
    int rc;
    mca_btl_ofi_module_t *ofi_btl = (mca_btl_ofi_module_t *) btl;
    mca_btl_ofi_endpoint_t *btl_endpoint = (mca_btl_ofi_endpoint_t *) endpoint;
    mca_btl_ofi_context_t *ofi_context;

    MCA_BTL_OFI_NUM_RDMA_INC(ofi_btl);

    ofi_context = get_ofi_context(ofi_btl);

    /* create completion context */
    mca_btl_ofi_rdma_completion_t *comp;
    comp = mca_btl_ofi_rdma_completion_alloc(btl, endpoint, ofi_context, local_address,
                                             local_handle, cbfunc, cbcontext, cbdata,
                                             MCA_BTL_OFI_TYPE_PUT);

    remote_address = (remote_address - (uint64_t) remote_handle->base_addr);

    /* Remote write data across the wire */
    rc = fi_write(ofi_context->tx_ctx, local_address, size, /* payload */
                  local_handle->desc, btl_endpoint->peer_addr, remote_address, remote_handle->rkey,
                  &comp->comp_ctx); /* completion context */

    if (-FI_EAGAIN == rc) {
        MCA_BTL_OFI_NUM_RDMA_DEC(ofi_btl);
        opal_free_list_return(comp->base.my_list, (opal_free_list_item_t *) comp);
        return OPAL_ERR_OUT_OF_RESOURCE;
    }

    if (0 != rc) {
        MCA_BTL_OFI_NUM_RDMA_DEC(ofi_btl);
        opal_free_list_return(comp->base.my_list, (opal_free_list_item_t *) comp);
        BTL_ERROR(("fi_write failed with %d:%s", rc, fi_strerror(-rc)));
        MCA_BTL_OFI_ABORT();
    }

    return OPAL_SUCCESS;
}
```


# btl init
```c
/** OFI btl component */
mca_btl_ofi_component_t mca_btl_ofi_component = {
    .super =
        {
            .btl_version =
                {
                    MCA_BTL_DEFAULT_VERSION("ofi"),
                    .mca_open_component = mca_btl_ofi_component_open,
                    .mca_close_component = mca_btl_ofi_component_close,
                    .mca_register_component_params = mca_btl_ofi_component_register,
                },
            .btl_data =
                {/* The component is not checkpoint ready */
                 .param_field = MCA_BASE_METADATA_PARAM_NONE},

            .btl_init = mca_btl_ofi_component_init,
            .btl_progress = mca_btl_ofi_component_progress,
        },
};

```

# ofi interface
这个 `enum mca_btl_ofi_hdr_type` 出现在 Open MPI 项目的 **ofi BTL 模块**（btl: byte transfer layer，用于点对点通信的模块）中，具体对应的是 OpenFabrics Interfaces (OFI) 网络抽象层。

这里一一解释各个枚举值的含义：

---

| 枚举值                      | 含义解释                                                          |
| ------------------------ | ------------------------------------------------------------- |
| `MCA_BTL_OFI_TYPE_PUT`   | **PUT 操作**，发起一方将数据直接写入远端内存                                    |
| `MCA_BTL_OFI_TYPE_GET`   | **GET 操作**，发起一方从远端内存直接读取数据                                    |
| `MCA_BTL_OFI_TYPE_AOP`   | **Atomic Operation (AOP)**，原子操作，如加法、最大值、最小值（对远端内存的原子更新）       |
| `MCA_BTL_OFI_TYPE_AFOP`  | **Atomic Fetch Operation (AFOP)**，带有返回值的原子操作，比如 fetch-and-add |
| `MCA_BTL_OFI_TYPE_CSWAP` | **Compare-and-Swap**，比较并交换操作（典型的同步原语，用于实现锁或一致性控制）             |
| `MCA_BTL_OFI_TYPE_SEND`  | **SEND 操作**，发送数据包（短消息或 eager 发送）                              |
| `MCA_BTL_OFI_TYPE_RECV`  | **RECV 操作**，接收数据包                                             |
| `MCA_BTL_OFI_TYPE_TOTAL` | 枚举的总数，用作数组大小或循环上限（它本身不是一种操作，而是枚举计数器）                          |

---

### 📌 背景理解

* 这些枚举定义的是 BTL 层在和 libfabric（OFI）交互时支持的不同操作类型。
* OFI 是 libfabric 提供的高性能网络接口，用于 RDMA、atomic、tagged messaging 等。
* 在 Open MPI 内部，这些枚举值用于标记当前要发出的 `ofi` 请求的 **类型**，然后根据类型选择不同的 libfabric API 调用（比如 `fi_write()`, `fi_read()`, `fi_send()`, `fi_atomic()` 等）。

---

### 🛠️ 用法示例

在 BTL 实现代码里，你可能会看到类似：

```c
switch (hdr->type) {
    case MCA_BTL_OFI_TYPE_PUT:
        // 调用 fi_write()
        break;
    case MCA_BTL_OFI_TYPE_GET:
        // 调用 fi_read()
        break;
    case MCA_BTL_OFI_TYPE_SEND:
        // 调用 fi_send()
        break;
    // 其他 case...
}
```

---

如果你需要：
✅ 分析代码中具体用这些枚举的地方
✅ 编写单元测试或 mock 这些类型
✅ 对比 RDMA vs SEND/RECV 在 ofi 层的实现

直接跟我说，我可以帮你写代码模板或解析调用栈。要试试吗？🌟



# ofi read/write vs send/recv
好问题！我帮你梳理一下 **在 OFI 层（libfabric）中 RDMA 操作 vs SEND/RECV 操作的核心区别和实现路径**。

---

### 🌊 **对比目标**

| 项目         | RDMA (PUT/GET)                        | SEND/RECV (消息传递)                         |
| ---------- | ------------------------------------- | ---------------------------------------- |
| **操作类型**   | 内存直接读写（Remote Memory Access）          | 消息式通信（Message Passing）                   |
| **典型 API** | `fi_write()`、`fi_read()`              | `fi_send()`、`fi_recv()`                  |
| **数据流向**   | 一方直接访问远端暴露的内存区域                       | 一方发送消息，另一方主动收消息                          |
| **连接要求**   | 需要 memory region 注册 + RMA 权限 + 地址/键信息 | 需要绑定 endpoint + 完成对等 handshake (连接型或无连接) |
| **同步性**    | 异步，但需要 memory fence / flush 保证可见性     | 按消息边界自动同步（到达即投递）                         |
| **常用场景**   | 大数据块传输、高性能一对一、零拷贝                     | 小消息、控制消息、tag-matching                    |

---

### 🔬 **OFI 层实现要点**

#### ✅ RDMA (PUT / GET)

* 涉及远端 memory region：

  * 使用 `fi_mr_reg()` 注册本地和远端内存
  * 远端提供 `fi_rma_iov` 或 `fi_addr` + rkey
* 发起端调用：

  * `fi_write()`：把本地 buffer 写到远端
  * `fi_read()`：从远端 buffer 读到本地
* 完成方式：

  * 使用 `fi_cq`（Completion Queue）或 `fi_cntr` 检查完成
* 高效之处：

  * 不需要远端 CPU 参与，只需网卡（RNIC/DPU）完成

---

#### ✅ SEND / RECV

* 发送端调用：

  * `fi_send()` 提交消息到发送队列
* 接收端调用：

  * `fi_recv()` 预先投递接收 buffer 到接收队列
* 完成方式：

  * 发送、接收双方都要在 `fi_cq` 上检查对应事件
* 特点：

  * 按消息边界保证完整性
  * 支持 tag-matching（如 `fi_tsend()` / `fi_trecv()`）

---

### 📊 **对比表（实现细节）**

| 细节         | RDMA                      | SEND/RECV                 |
| ---------- | ------------------------- | ------------------------- |
| 内存注册       | 必须（local & remote）        | 只需本地 buffer 注册            |
| 内存地址信息     | 需要远端地址和 rkey              | 不需要，只需远端 endpoint address |
| 通信建立       | 需要 QP 和 RMA 权限            | 需要 endpoint 和 handshake   |
| 互操作要求      | 远端不需要显式配合                 | 双方需投递匹配的 send/recv        |
| 延迟 vs 吞吐   | 低延迟、超高吞吐（大块传输）            | 较高延迟、适合小消息                |
| 硬件 offload | RDMA 专用网卡（RNIC）直接 offload | 通用网络接口、可软件实现              |

---

### 🔩 **代码示例：两者调用对比**

✅ RDMA PUT:

```c
struct iovec local_iov = { .iov_base = local_buf, .iov_len = size };
struct fi_rma_iov remote_iov = { .addr = remote_addr, .key = rkey, .len = size };

ret = fi_write(ep, &local_iov, 1, desc, remote_addr, rkey, context);
```

✅ SEND/RECV:

```c
// Sender
ret = fi_send(ep, send_buf, size, desc, remote_addr, context);

// Receiver
ret = fi_recv(ep, recv_buf, size, desc, remote_addr, context);
```

---

### 🚀 总结

✅ RDMA 适合：

* 大数据块传输
* 不需要远端 CPU 参与的场景（比如 GPU 直连、内存拷贝）

✅ SEND/RECV 适合：

* 控制消息
* 需要可靠投递或 tag 匹配的消息流

---

如果你想要：

* 写一个 **Open MPI + ofi 的小 demo**
* 看一段 **btl/ofi 模块中 PUT 和 SEND 分支的代码解析**
* 或者用 `perf`、`strace` 实际抓调用栈

直接跟我说，我可以帮你写出来！要试试吗？💡





