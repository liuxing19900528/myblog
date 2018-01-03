马哥私房菜 https://shop592330910.taobao.com/

自动abandon很久之前的处于open的patch

```

GERRIT_SERVER=
for i in $(ssh $GERRIT_SERVER gerrit query status:open --current-patch-set age:2month | grep revision: | awk '{print $2}')
do
    LAST_REVISION=$i
    ABANDON_PROJECT=$(ssh $GERRIT_SERVER gerrit query status:open  $LAST_REVISION | grep -E "^\s+project:" | awk '{ print $2 }' | head -1)
    echo "==================="
    echo "Gerrit Server: $GERRIT_SERVER"
    echo "Project: $ABANDON_PROJECT"
    echo "Abandon revision: $LAST_REVISION"
    ssh $GERRIT_SERVER gerrit review --project $ABANDON_PROJECT --abandon $LAST_REVISION
    echo "==================="
    echo
done

```
