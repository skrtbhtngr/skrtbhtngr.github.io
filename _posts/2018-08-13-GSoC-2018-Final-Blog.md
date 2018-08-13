#### Project: Automatic freeing of resource
#### Summary: ​Implement \_\_attribute\_\_((cleanup))​ for libvirt
#### Organization: libvirt
#### Mentors: Erik Skultety and Pavel Hrdina

**TL;DR:** [Here](https://libvirt.org/git/?p=libvirt.git&a=search&h=HEAD&st=author&s=Sukrit+Bhatnagar) is the work I did.

## Introduction
In the libvirt core C library, goto jumps are used frequently. Contrary to the popular negative notion :grey_exclamation:, they serve a [special purpose](https://libvirt.org/hacking.html#goto) here. They are employed to perform cleanup tasks like freeing memory allocated to pointers, closing file handles, unlocking mutexes, unref-ing objects etc. before a function returns. But, in most cases, it led to functions with too many jumps, all for just one task -- freeing up memory!

The [idea](https://wiki.libvirt.org/page/Google_Summer_of_Code_Ideas#Automatic_freeing_of_memory), as suggested by Daniel, was to use GNU C's cleanup attribute. A more detailed description can be found in my project proposal [here]().

After a few (long) mailing list discussions <sup>[[1](https://www.redhat.com/archives/libvir-list/2018-March/msg01532.html)]</sup><sup>[[2](https://www.redhat.com/archives/libvir-list/2018-June/msg00807.html)]</sup> and weekly meetings with Erik and Pavel, we all agreed upon the use of macros and their design.


## Macro Design

The macro design is primarily inspired from [GLib](https://github.com/GNOME/glib/blob/f92359b593b9eb72cea2a67fcfe01b94520dce5a/glib/gmacros.h).

To implement the automatic cleanup functionality in the code, a set of four macros were introduced in `src/util/viralloc.h`.
<br>

{% highlight C %}
# define VIR_AUTOFREE(type) __attribute__((cleanup(virFree))) type
{% endhighlight %}

This macro is used to declare pointers to scalar types (int, char etc.) and external types (struct nlmsghdr etc.). The function `virFree` will be called automatically on those variables.
<br>

{% highlight C %}
# define VIR_AUTOPTR_FUNC_NAME(type) type##AutoPtrFree

# define VIR_DEFINE_AUTOPTR_FUNC(type, func) \
    static inline void VIR_AUTOPTR_FUNC_NAME(type)(type **_ptr) \
    { \
        if (*_ptr) \
            (func)(*_ptr); \
        *_ptr = NULL; \
    } \

# define VIR_AUTOPTR(type) \
    __attribute__((cleanup(VIR_AUTOPTR_FUNC_NAME(type)))) type *
{% endhighlight %}

These macros are meant for libvirt types such as virBitmap and virHashTable. The usage of these macros to implement the cleanup functionality is two-fold:

1. Define a new Free wrapper using `VIR_DEFINE_AUTOPTR_FUNC`.
    
    Pass a libvirt datatype `type` and a pre-defined Free function `func` to it. This macro is usually invoked in the corresponding `.h` file of a module for an externally visible function `func`, or in the corresponding `.c` file for a static function `func`.
2. Declare pointers to libvirt datatype `type` which have to be auto-freed upon function return using the VIR_AUTOPTR macro.



## Work Done

More than 100 module files were modified in the util directory to implement the cleanup functionality. The changes made to a file followed an order, each of which had its own patch.

1. define a Free function for a libvirt type (if not already done)
2. define a Free wrapper using `VIR_DEFINE_AUTOPTR_FUNC` in header/source file
3. use `VIR_AUTOFREE` in the source file
4. use `VIR_AUTOPTR` in the source file

There were many files where a subset of these steps was not needed.

A new type `virString` was typedef'd for `char *` for ease of use with the cleanup macros.

A new syntax check rule was added in `cfg.mk` to ensure that a variable declared using the macros is initialized to NULL.

[Here](https://libvirt.org/git/?p=libvirt.git&a=search&h=HEAD&st=author&s=Sukrit+Bhatnagar) are all my patches that were accepted into the master branch.

There were some patches which were not pushed upstream due to time constraints. They can be found in my (forked) libvirt on Github in the branch [gsoc-2018](https://github.com/skrtbhtngr/libvirt/tree/gsoc-2018). The pending patches are [here](https://github.com/libvirt/libvirt/compare/master...skrtbhtngr:gsoc-2018).


## Results

The changes made in the past 3 months mostly involved introducing new Free wrappers and modifying pointer declarations to use the macros. As a result of these changes, a great deal of LOC (lines of code), as well as goto jumps, were reduced.

Following are two examples showing the usage of these macros which I find worth mentioning. The code pieces on the left are the original versions, and the code pieces on the right are the versions after the macros are used.


#### An Example: `virISCSIRescanLUNs` in viriscsi.c

{% highlight C %}
int                                                     |   int
virISCSIRescanLUNs(const char *session)                 |   virISCSIRescanLUNs(const char *session)
{                                                       |   {
    virCommandPtr cmd = NULL;                           |       VIR_AUTOPTR(virCommand) cmd = NULL;
    cmd = virCommandNewArgList(ISCSIADM, "--mode",      |       cmd = virCommandNewArgList(ISCSIADM, "--mode",
                               "session", "-r",         |                                  "session", "-r",
                               session, "-R", NULL);    |                                  session, "-R", NULL);
    int ret = virCommandRun(cmd, NULL);                 |       return virCommandRun(cmd, NULL);
    virCommandFree(cmd);                                |   }
    return ret;                                         |
}                                                       |
{% endhighlight %}

Introducing VIR_AUTOPTR in this function simplifies the code flow. As the function ` virCommandFree()` will be called implicitly, the variable `ret` can now be dropped.


#### Another Example: `virNetDevIPAddrAdd` in virnetdevip.c

{% highlight C %}
    virSocketAddr *broadcast = NULL;                    |     unsigned int recvbuflen;
    int ret = -1;                                       |     VIR_AUTOPTR(virNlMsg) nlmsg = NULL;
    struct nl_msg *nlmsg = NULL;                        |     VIR_AUTOPTR(virSocketAddr) broadcast = NULL;
    struct nlmsghdr *resp = NULL;                       |     VIR_AUTOFREE(struct nlmsghdr *) resp = NULL;
    unsigned int recvbuflen;                            |     VIR_AUTOFREE(char *) ipStr = NULL;
    char *ipStr = NULL;                                 |     VIR_AUTOFREE(char *) peerStr = NULL;
    char *peerStr = NULL;                               |     VIR_AUTOFREE(char *) bcastStr = NULL;
    char *bcastStr = NULL;                              |
...                                                     | ...
    ret = 0;                                            |     return 0;
  cleanup:                                              |
     VIR_FREE(ipStr);                                   |
     VIR_FREE(peerStr);                                 |
     VIR_FREE(bcastStr);                                |
     nlmsg_free(nlmsg);                                 |
     VIR_FREE(resp);                                    |
     VIR_FREE(broadcast);                               |
     return ret;                                        |     
{% endhighlight %}

Using the macros for 6 variables here, the whole cleanup section as well as the varible `ret` can be dropped. All the goto jumps to `cleanup` can be replaced with return statements.


#### Yet Another Example: `virNetDevGetVirtualFunctions` in virnetdev.c

{% highlight C %}
    int ret = -1;                                       |     size_t i;
    size_t i;                                           |     VIR_AUTOFREE(char *) pf_sysfs_device_link = NULL;
    char *pf_sysfs_device_link = NULL;                  |     VIR_AUTOFREE(char *) pci_sysfs_device_link = NULL;
    char *pci_sysfs_device_link = NULL;                 |     VIR_AUTOFREE(char *) pciConfigAddr = NULL;
    char *pciConfigAddr = NULL;                         |     VIR_AUTOFREE(char *) pfPhysPortID = NULL;
    char *pfPhysPortID = NULL;                          |     VIR_AUTOFREE(char **) tmpVfname = NULL;
                                                        |     VIR_AUTOFREE(virPCIDeviceAddressPtr *) tmpVirtFns = NULL;
...                                                     | ...
    ret = 0;                                            |     VIR_STEAL_PTR(*vfname, tmpVfname);
                                                        |     VIR_STEAL_PTR(*virt_fns, tmpVirtFns);
 cleanup:                                               |
    if (ret < 0) {                                      |     return 0;
        VIR_FREE(*vfname);                              |
        VIR_FREE(*virt_fns);                            |
    }                                                   |
    VIR_FREE(pfPhysPortID);                             |
    VIR_FREE(pf_sysfs_device_link);                     |
    VIR_FREE(pci_sysfs_device_link);                    |
    VIR_FREE(pciConfigAddr);                            |
    return ret;                                         |
{% endhighlight %}

Here, `vfname` and `virt_fns` are parameters passed to this function. In the case of functions like this one, where the parameters have to be freed in the cleanup section, a different approach is used. Dummy variables are added, one for each such parameter, they replace all occurrences of the said parameters in the function code.

In the end, when the function follows the "success" path, the values in those dummy variables will be moved to their corresponding parameters. Those dummy variables will then be NULL and thus their values (now in the parameters) are safe from `virFree`.

Since the whole cleanup section can be discarded, all goto jumps can be replaced by return statements. In this case, when the function takes the "cleanup" path, the parameters will be untouched, and the values in those dummy variables will be freed automatically, which is what we desire.


## Challenges faced
Here are some of the major issues I faced during the course of this project:

* This project involved sending a lot of patches :sweat_smile:, mainly due to the fact that each logically similar change to a file should be in a separate patch. Furthermore, there had to be an order among those patches. For instance, for a file virxyz.c, there must be first a patch for defining new Free wrapper using VIR\_DEFINE\_AUTOPTR\_FUNC, then a  patch following it for using VIR\_AUTOFREE, then a patch following it for using VIR\_AUTOPTR. In many instances, the changes in those patches had to be fixed. This led to a lot of rebasing and conflict resolving which was pure pain.

* In implementing the cleanup attribute, almost all of the source files in the library have to be touched. To make such broad changes, we had a few community discussions regarding the design of the macros. These discussions, while essential, consumed a few weeks of my coding period.


## Some tips for using the macros

* If a variable has to retain a value on success, but freed on other paths, assign NULL to it before returning from the function on the success path. This will ensure that the value to be returned is not auto-freed after the function returns.

    For example, the function `virHashAddEntry` adds user data to a hash table upon success and returns 0. It returns -1 upon error, in which case the caller may free the user data. If the variable containing the user data is declared using the macros, it must be ensured that the variable is set to NULL if `virHashAddEntry ` succeeds, so that the user data added to the hash table is not accidentally freed.

* If a function parameter has to be freed on other paths, but not on the success path, use a dummy local variable in its place all over the function and steal dummy's value into the parameter before the function returns on the success path. This will ensure that the parameter is not assigned any value upon paths other than "success" and the dummy's value, which was to be assigned to it, will be freed automatically. See the `virNetDevGetVirtualFunctions` example in a section below.

## Work left to be done
Here are a few things that are left to be done on top of my work:

[ ] Change all the Free function signatures to accept a double pointer
[ ] Change all the ListFree helper functions to take a double pointer.
[ ] Implement VIR\_AUTOCLOSE
[ ] Implement VIR\_AUTOCLEAN