---
layout: post
title:  "Programaticly configuring VMware Storage Profile API with Python"
date:   2020-08-12 20:20:56 +0100
tags:
    - DevOps
permalink: python-pbm/
---

I was looking to programaticly configuring VMware Storage Profile recently. My scripting language of choice is Python, VMware maintain [pyVmomi](https://github.com/vmware/pyvmomi) is which is the Python SDK for the VMware vSphere API. pyVmomi can be used to form binding and configure the [vSphere Web Services API](https://code.vmware.com/apis/968/vsphere) and the [VMware Storage Policy API](https://code.vmware.com/apis/971).

The Storage Policy API is exposed as a Web service, running on VMware vCenter server systems.  When I approached the requirement it wasn't clear to me how to connect to the Storage Policy API Web service. I read the [VMware Storage Policy SDK Programming Guide](https://code.vmware.com/docs/11900/vmware-storage-policy-sdk-programming-guide) section on forming connection which describes using the session cookie from a vCenter Server session to establish the Storage Policy session.

pyVim is a client-side Python API which wraps pvVmomi these are made available as a package.  These are published to PyPI repository and as such can be installed easily with pip.

```
pip install --upgrade pyvmomi
```

It we look at [pyVmomi in PyDoc](https://pydoc.net/pyvmomi/) we can see that pyVim.connect is a function SmartConnect which can be used to form connection to the vSphere Web Services API.

```
#!/usr/bin/env python3
# Connect to a VMOMI ServiceInstance.

import ssl argparse atexit getpass
from pyVim.connect import SmartConnect

def get_args():
    parser = argparse.ArgumentParser(
        description='Arguments for talking to vCenter')

    parser.add_argument('-s', '--host',
                        required=True,
                        action='store',
                        help='vCenter Server FQDN or IP address')

    parser.add_argument('-o', '--port',
                        type=int,
                        default=443,
                        action='store',
                        help='vCenter Server TCP port')

    parser.add_argument('-u', '--user',
                        required=True,
                        action='store',
                        help='Username to login to vCenter Server')

    parser.add_argument('-p', '--password',
                        required=False,
                        action='store',
                        help='Password to login to vCenter Server')

    args = parser.parse_args()

    if not args.password:
        args.password = getpass.getpass(prompt='Enter password:')

    return args

def main():
    args = get_args()

    """ connect to vcenter service instance """

    context = None
    if hasattr(ssl, "_create_unverified_context"):
        context = ssl._create_unverified_context()

    serviceInstance = SmartConnect(
                    host=args.host,
                    user=args.user,
                    pwd=args.password,
                    port=args.port,
                    sslContext=context)

    atexit.register(Disconnect, serviceInstance)
```

To connect to the VMware Storage Policy API we need to get the session cookie. The VmomiSupport provides us helper functions to do things like gather the context which includes session cookie details. With this we can form a stub session to the Storage Policy API.

```
    """ connect to pbm service instance """

    VmomiSupport.GetRequestContext()["vcSessionCookie"] = \
    serviceInstance._stub.cookie.split('"')[1]
    hostname = serviceInstance._stub.host.split(":")[0]
    pbmStub = SoapStubAdapter(
        host=hostname,
        version="pbm.version.version1",
        path="/pbm/sdk",
        poolSize=0,
        sslContext=ssl._create_unverified_context())
    pbmServiceInstance = pbm.ServiceInstance("ServiceInstance", pbmStub)
```

With this connection in place we can make our request, in this example pull back all of the Storage Profiles.

```

    """ get profiles """

    profileManager = pbmServiceInstance.RetrieveContent().profileManager
    profiles = profileManager.PbmQueryProfile(resourceType=pbm.profile.ResourceType(resourceType="STORAGE"))
    print(profiles)

if __name__ == '__main__':
    main()
```
