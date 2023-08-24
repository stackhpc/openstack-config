# Automatic Template Generation for CAPI Driver 

1. `git cherry-pick XXXXXX` if necessary (will replace this with final commit hash before this PR is merged)
2. change `openstack_images` list to `glance images` in `etc/openstack/openstack-config.yml` if necessary
3. ensure that `openstack_images` looks like: 
```
# Images to be uploaded 
openstack_images: "{{ glance_images + kubernetes_images }}"
```
4. source the openstack-config venv you have set up
5. source the rc.sh file that points to the appropriate cloud, or provide a `clouds.yaml` under `./tools/merge_config`
6. ensure `wget`, `python-octaviaclient` and `python-magnumclient` are installed
7. If you have existing `openstack_container_clusters_templates` defined, move them to `etc/openstack/container-clusters.yml`
8. run `./tools/merge_config/bin/run` 
9. check output at `etc/openstack-config/container-clusters.yml`
10. If it all checks out, run: 
```
tools/openstack-config -p ansible/openstack-container-clusters.yml -- --vault-password-file ~/.vault-secret -e@etc/openstack-config/container-clusters.yml
```
Or include the same `-e@etc/openstack-config/container-clusters.yml` when you run the entire openstack-config suite. 

This must be ran before `os-images` is, if any kubernetes images/templates are being hidden/retired, because the cluster template cannot be hidden after the corresponding image is hidden.

If you run the above command, you can run: 
```
tools/openstack-config -p ansible/openstack-images.yml -- --vault-password-file ~/.vault-secret  -e@etc/openstack-config/container-clusters.yml
```
afterwards, to upload/hide any of the kubernetes images. 

Note: If you run out of space to store images on the control host, you may need to run this in sections - comment out blocks of images, making sure to remove any cached images under `~/openstack-config/ansible/openstack-config-image-cache/` before moving onto the next block.

Note: If the image cache has been cleared (i.e. the old images being set to hidden no longer exist in the cache), then `openstack.cloud.image` will not recognise that it is the same image and will upload a new one, so you should comment these images out from the list. 

Note: Container template hiding does not no-op, due to the current state of the magnum API. If the template is already hidden and you rerun this, you will get the error `ClusterTemplate 4ce13776-da1e-42c9-b6f3-1249363d7a4e is referenced by one or multiple clusters (HTTP 400)`
You may also get an error like `Image ubuntu-focal-kube-v1 (HTTP 400)` , which is because the image has been hidden. If your template is already hidden, this is not a problem, but if it isn't hidden, you will need to unhide the images before the template hiding can run. 