### Intelij IDEA

####  RunDashBoard

View -> Tool Windows - run Dashboard

如果没有找到的话，找到当前项目 隐藏文件夹 .idea 目录下 workspace.xml 文件，添加以下内容

```xml
<component name="RunDashboard">
    <option name="configurationTypes">
      <set>
        <option value="SpringBootApplicationConfigurationType" />
      </set>
    </option>
    <option name="ruleStates">
      <list>
        <RuleState>
          <option name="name" value="ConfigurationTypeDashboardGroupingRule" />
        </RuleState>
        <RuleState>
          <option name="name" value="StatusDashboardGroupingRule" />
        </RuleState>
      </list>
    </option>
  </component>
```

