# Save / restore

## VM

Create a VM.

```shell
# Replace the path in vm.xml with an absolute one and create the VM.
sed "s|\$IMAGES|$(pwd)/images|g" vm.xml | virsh create /dev/stdin

Domain 'vm' created from /dev/stdin
```

```shell
virsh list --all

 Id   Name   State
----------------------
 1    vm     running
```

Create a snapshot of the VM.

```shell
mkdir -p $PWD/snapshots
virsh dumpxml --migratable --domain vm > snapshots/snapshot.xml
virsh save --domain vm --xml snapshots/snapshot.xml --file snapshots/snapshot.saved

Domain 'vm' saved to snapshots/snapshot.saved
```

Check the VM status.

```shell
virsh list --all

 Id   Name   State
----------------------
```

Restore the snapshot.

```shell
virsh restore --xml snapshots/snapshot.xml --file snapshots/snapshot.saved --running

Domain restored from snapshots/snapshot.saved
```

Check the VM status.

```shell
virsh list --all

 Id   Name   State
----------------------
 2    vm     running
```

Connect to the VM.

```shell
virt-viewer
```

Cleanup.

```shell
virsh destroy vm
```

## Docker

Setup Docker:

```shell
cat <<EOF > /etc/docker/daemon.json
{
  "experimental": true
}
EOF

sudo systemctl restart docker
```

Create snapshot:

```shell
docker run -di --name ubuntu ubuntu

docker exec -ti ubuntu bash
# -> apt update && apt install -y tmux vim
# -> tmux
# -> tmux set -g status off
# -> vim
# -> type something in buffer
# -> detach tmux (ctrl+b d)
# -> exit container (ctrl + d)

mkdir -p $PWD/checkpoints
docker checkpoint create --leave-running --checkpoint-dir $PWD/checkpoints ubuntu first
docker stop ubuntu
```

Check.

```shell
docker ps

CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```shell
docker checkpoint ls --checkpoint-dir $PWD/checkpoints ubuntu

CHECKPOINT NAME
first
```

Restore snapshot:

```shell
# Work around a docker bug https://github.com/moby/moby/issues/37344
cp -R $PWD/checkpoints/first /var/lib/docker/containers/$(docker ps -aq --no-trunc --filter name=ubuntu)/checkpoints/

docker start --checkpoint first ubuntu

docker exec -ti ubuntu tmux a
```

The state is exactly the same as we left it before the checkpoint/restore.
