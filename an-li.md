### 案例对比（dva和react.eval）

一个部门，有4个同事，1个测试员，一个主管，某天，测试员要和主管请假,主管需要知道请假人的名字和请假时间

**dva版本**

同事

![](/assets/importworkmate.png)

主管

![](/assets/importManager.png)

测试人员

![](/assets/importtest.png)

model

![](/assets/importmodel.png)

部门

![](/assets/importDep.png)

最终效果

![](/assets/tshow.gif)

**这样就实现了 测试人员  向 部门主管请假的目的\(dva版\)。**

---------------------------------------------------------------------------------------------

如果此时 ，测试人员的直接上级发生了变更，其他人向原来的主管请假，测试人员向测试主管请假









