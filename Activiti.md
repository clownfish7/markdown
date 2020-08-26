### Activiti BpmnModel

```
package cn.ponytech.test;

import cn.ponytech.listener.JudgeFalseListener;
import org.activiti.bpmn.converter.BpmnXMLConverter;
import org.activiti.bpmn.model.Process;
import org.activiti.bpmn.model.*;
import org.activiti.validation.ProcessValidator;
import org.activiti.validation.ProcessValidatorFactory;
import org.activiti.validation.ValidationError;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @author yzy
 * @classname BpmnTest
 * @description TODO
 * @create 2020-05-11 14:01
 */
public class BpmnTest {

    @Test
    public void testBpmn() {

        // 创建BpmnModel对象
        BpmnModel bpmnModel = createBpmnModel();

        // 创建process节点 并添加至process, <process id="myProcess" name="My process" isExecutable="true">
        Process process = createProcess();
        bpmnModel.addProcess(process);

        // 创建开始结束节点 并添加至process
        process.addFlowElement(createStartEvent());
        process.addFlowElement(createEndEvent());

        // 创建任务节点 并添加至process, <userTask id="usertask1" name="提交审批"/>
        UserTask userTask1 = createUserTask("usertask1", "提交审批");
        UserTask userTask2 = createUserTask("usertask2", "总经理审批");
        UserTask userTask3 = createUserTask("usertask3", "部门经理审批");

        process.addFlowElement(userTask1);
        process.addFlowElement(userTask2);
        process.addFlowElement(userTask3);

        // 创建排他网关 并添加至process, <exclusiveGateway id="exclusivegateway1" name="Exclusive Gateway"/>
        ExclusiveGateway exclusiveGateway = createExclusiveGateway("exclusivegateway1", "Exclusive Gateway");
        process.addFlowElement(exclusiveGateway);

        // 创建连线 并添加至process, <sequenceFlow id="flow1" sourceRef="startevent1" targetRef="usertask1"/>
        SequenceFlow sequenceFlow1 = createSequenceFlow("flow1", "", "start", "usertask1", "", process);
        process.addFlowElement(sequenceFlow1);

        // 创建连线 并添加至process, <sequenceFlow id="flow2" sourceRef="usertask1" targetRef="exclusivegateway1"/>
        SequenceFlow sequenceFlow2 = createSequenceFlow("flow2", "", "usertask1", "exclusivegateway1", "", process);
        process.addFlowElement(sequenceFlow2);

        // 创建连线 并添加至process
        // <sequenceFlow id="flow3" name="大于三天" sourceRef="exclusivegateway1" targetRef="usertask2">
        //    <conditionExpression xsi:type="tFormalExpression"><![CDATA[${day>3}]]></conditionExpression>
        // </sequenceFlow>
        SequenceFlow sequenceFlow3 = createSequenceFlow("flow3", "大于三天", "exclusivegateway1", "usertask2", "${day>3}", process);
        process.addFlowElement(sequenceFlow3);

        // 创建连线 并添加至process
        // <sequenceFlow id="flow4" name="小于三天" sourceRef="exclusivegateway1" targetRef="usertask3">
        //   <conditionExpression xsi:type="tFormalExpression"><![CDATA[${day<=3}]]></conditionExpression>
        // </sequenceFlow>
        SequenceFlow sequenceFlow4 = createSequenceFlow("flow4", "小于三天", "exclusivegateway1", "usertask3", "${day<=3}", process);
        process.addFlowElement(sequenceFlow4);

        // 创建连线 并添加至process
        // <sequenceFlow id="flow6" sourceRef="usertask2" targetRef="endevent2"></sequenceFlow>
        // <sequenceFlow id="flow7" sourceRef="usertask3" targetRef="endevent2"></sequenceFlow>
        SequenceFlow sequenceFlow6 = createSequenceFlow("flow6", "", "usertask2", "end", "", process);
        SequenceFlow sequenceFlow7 = createSequenceFlow("flow7", "", "usertask3", "end", "", process);
        process.addFlowElement(sequenceFlow6);
        process.addFlowElement(sequenceFlow7);

        // 为排他网关添加出线
        List<SequenceFlow> outgoingFlows = new ArrayList<>();
        outgoingFlows.add(sequenceFlow3);
        outgoingFlows.add(sequenceFlow4);
        exclusiveGateway.setOutgoingFlows(outgoingFlows);

        // validate the bpmnModel
        bpmnModelValidate(bpmnModel);


        BpmnXMLConverter bpmnXMLConverter = new BpmnXMLConverter();
        byte[] bytes = bpmnXMLConverter.convertToXML(bpmnModel);

        System.out.println(new String(bytes));

        bpmnModel.getProcesses().get(0).findFlowElementsOfType(UserTask.class).forEach(System.out::println);

//        bpmnModel.getMainProcess().
    }

    /**
     * 创建 BpmnModel
     */
    private BpmnModel createBpmnModel() {
        return new BpmnModel();
    }

    /**
     * 创建 Process
     * <process id="myProcess" name="My process" isExecutable="true">
     * default isExecutable = true
     */
    private Process createProcess() {
        Process process = new Process();
        process.setId("myProcess");
        process.setName("MyProcess");
        return process;
    }

    /**
     * 创建 StartEvent
     * <startEvent id="start" name="开始"/>
     */
    private StartEvent createStartEvent() {
        StartEvent startEvent = new StartEvent();
        startEvent.setId("start");
        startEvent.setName("开始");
        return startEvent;
    }

    /**
     * 创建 EndEvent
     * <endEvent id="end" name="结束"/>
     */
    private EndEvent createEndEvent() {
        EndEvent endEvent = new EndEvent();
        endEvent.setId("end");
        endEvent.setName("结束");
        return endEvent;
    }

    /**
     * 创建连线
     */
    private SequenceFlow createSequenceFlow(String id, String name, String sourceRef, String tartgetRef, String conditionExpression, Process process) {
        SequenceFlow sequenceFlow = new SequenceFlow(sourceRef, tartgetRef);
        sequenceFlow.setId(id);
        if (StringUtils.isNotBlank(name)) {
            sequenceFlow.setName(name);
        }
        sequenceFlow.setSourceFlowElement(process.getFlowElement(sourceRef));
        sequenceFlow.setTargetFlowElement(process.getFlowElement(tartgetRef));
        if (StringUtils.isNotBlank(conditionExpression)) {
            sequenceFlow.setConditionExpression(conditionExpression);
        }
        ActivitiListener activitiListener = new ActivitiListener();
        activitiListener.setInstance(new JudgeFalseListener());
        activitiListener.setEvent("end");
        activitiListener.setImplementation("${asd}");
        activitiListener.setImplementationType("delegateExpression");
        sequenceFlow.setExecutionListeners(Arrays.asList(activitiListener));
        return sequenceFlow;
    }

    /**
     * 创建 Listener
     */
    private List<ActivitiListener> createActivitiListener() {
        ActivitiListener activitiListener = new ActivitiListener();
        activitiListener.setInstance(new JudgeFalseListener());
        activitiListener.setEvent("end");
        activitiListener.setImplementation("${asd}");
        activitiListener.setImplementationType("delegateExpression");
//        sequenceFlow.setExecutionListeners(Arrays.asList(activitiListener));
        return Arrays.asList(activitiListener);
    }

    /**
     * 创建 ExclusiveGateway
     */
    private ExclusiveGateway createExclusiveGateway(String id, String name) {
        ExclusiveGateway exclusiveGateway = new ExclusiveGateway();
        exclusiveGateway.setId(id);
        exclusiveGateway.setName(name);
        return exclusiveGateway;
    }

    /**
     * 创建 ParallelGateway
     */
    private ParallelGateway createParallelGateway(String id, String name) {
        ParallelGateway parallelGateway = new ParallelGateway();
        parallelGateway.setId(id);
        parallelGateway.setName(name);
        return parallelGateway;
    }

    /**
     * 创建 InclusiveGateway
     */
    private InclusiveGateway createInclusiveGateway(String id, String name) {
        InclusiveGateway inclusiveGateway = new InclusiveGateway();
        inclusiveGateway.setId(id);
        inclusiveGateway.setName(name);
        ExtensionElement extensionElement = new ExtensionElement();
//        extensionElement.addExtensionElement();
//        extensionElement.addChildElement();
        inclusiveGateway.addExtensionElement(extensionElement);
        return inclusiveGateway;
    }

    /**
     * 创建 UserTask
     */
    private UserTask createUserTask(String id, String name) {
        UserTask userTask = new UserTask();
        userTask.setId(id);
        userTask.setName(name);
        return userTask;
    }

    /**
     * 校验 bpmnModel
     * 模型校验器：https://www.jianshu.com/p/40e2d761477c
     */
    private void bpmnModelValidate(BpmnModel bpmnModel) {
        ProcessValidatorFactory processValidatorFactory = new ProcessValidatorFactory();
        ProcessValidator processValidator = processValidatorFactory.createDefaultProcessValidator();
        List<ValidationError> validate = processValidator.validate(bpmnModel);
        System.out.println(validate.size());
        if (validate.size() > 0) {
            for (ValidationError error : validate) {
                System.out.println(error.getProblem());
                System.out.println(error.isWarning());
            }
        }
    }
}

```

