**AWS Auto Scaling and Terraform Implementation**

## Introduction
This lecture covers **Auto Scaling in AWS**, explaining how **Auto Scaling Groups (ASGs)** work and how they can be configured to dynamically **scale EC2 instances** based on usage patterns.

### Resources
For reference, the Terraform code for this demo can be found in the `demo-15` folder.

---

## **1. Understanding AWS Auto Scaling**
AWS Auto Scaling enables you to automatically **add or remove EC2 instances** based on defined policies. This helps to:
- Maintain **optimal performance** during peak periods.
- **Reduce costs** by de-provisioning instances when they are not needed.
- **Improve availability** by ensuring instances are automatically replaced in case of failure.

### **1.1 Key Components**
To configure Auto Scaling, you need to set up **two main resources**:

#### **1.1.1 AWS Launch Configuration**
Defines the instance specifications:
- **AMI ID**: The image that will be used to launch instances.
- **Instance Type**: Specifies compute resources (e.g., `t2.micro`).
- **Security Groups**: Defines network rules.
- **Key Pair**: For SSH access.

#### **1.1.2 Auto Scaling Group (ASG)**
Manages the **actual scaling** of instances:
- **Minimum and Maximum Instances**: Defines the range of instances in the group.
- **VPC Subnets**: Specifies which subnets to launch instances in.
- **Health Checks**: Ensures instances remain operational.

---

## **2. Setting Up Auto Scaling in Terraform**

### **2.1 Launch Configuration**
```hcl
resource "aws_launch_configuration" "example-launchconfig" {
  name_prefix     = "example-launchconfig"
  image_id        = var.AMIS[var.AWS_REGION]
  instance_type   = "t2.micro"
  key_name        = aws_key_pair.mykeypair.key_name
  security_groups = [aws_security_group.allow-ssh.id]
}
```

### **2.2 Auto Scaling Group (ASG)**
```hcl
resource "aws_autoscaling_group" "example-autoscaling" {
  name                      = "example-autoscaling"
  vpc_zone_identifier       = [aws_subnet.main-public-1.id, aws_subnet.main-public-2.id]
  launch_configuration      = aws_launch_configuration.example-launchconfig.name
  min_size                  = 1
  max_size                  = 2
  health_check_grace_period = 300
  health_check_type         = "EC2"
  force_delete              = true
}
```

---

## **3. Implementing Auto Scaling Policies**
AWS Auto Scaling requires **CloudWatch Alarms** to monitor resource usage and trigger scaling policies.

### **3.1 Scaling Up Policy**
```hcl
resource "aws_autoscaling_policy" "example-cpu-policy" {
  name                   = "example-cpu-policy"
  autoscaling_group_name = aws_autoscaling_group.example-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}
```

### **3.2 Scaling Down Policy**
```hcl
resource "aws_autoscaling_policy" "example-cpu-policy-scaledown" {
  name                   = "example-cpu-policy-scaledown"
  autoscaling_group_name = aws_autoscaling_group.example-autoscaling.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = "-1"
  cooldown               = "300"
  policy_type            = "SimpleScaling"
}
```

### **3.3 CloudWatch Alarm for Scaling Up**
```hcl
resource "aws_cloudwatch_metric_alarm" "example-cpu-alarm" {
  alarm_name          = "example-cpu-alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "30"
  alarm_actions       = [aws_autoscaling_policy.example-cpu-policy.arn]
}
```

### **3.4 CloudWatch Alarm for Scaling Down**
```hcl
resource "aws_cloudwatch_metric_alarm" "example-cpu-alarm-scaledown" {
  alarm_name          = "example-cpu-alarm-scaledown"
  comparison_operator = "LessThanOrEqualToThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "5"
  alarm_actions       = [aws_autoscaling_policy.example-cpu-policy-scaledown.arn]
}
```

---

## **4. Demo: Deploying Auto Scaling with Terraform**

1. **Initialize Terraform:**
   ```bash
   terraform init
   ```
2. **Apply the Terraform Configuration:**
   ```bash
   terraform apply
   ```
3. **Verify in AWS Console:**
   - Check **Auto Scaling Groups**.
   - View instances created.
   - Check **CloudWatch Metrics** for CPU usage.

4. **Generate Load for Scaling Up:**
   ```bash
   sudo apt install stress
   stress --cpu 2 --timeout 300
   ```
5. **Observe Scaling Actions:**
   - Auto Scaling should create a new instance.
   - After 5 minutes, the instance count should return to normal when CPU usage drops.

6. **Destroy the Resources After Testing:**
   ```bash
   terraform destroy
   ```

---

## **5. Notifications via AWS SNS (Optional)**
To receive alerts when Auto Scaling events occur, use **AWS SNS**:

```hcl
resource "aws_sns_topic" "example-sns" {
  name         = "asg-alerts"
  display_name = "Auto Scaling Notifications"
}
```

Attach it to the ASG:
```hcl
resource "aws_autoscaling_notification" "example-notify" {
  group_names = [aws_autoscaling_group.example-autoscaling.name]
  topic_arn   = aws_sns_topic.example-sns.arn
  notifications = ["autoscaling:EC2_INSTANCE_LAUNCH", "autoscaling:EC2_INSTANCE_TERMINATE"]
}
```

---

## **6. Summary**
- **AWS Auto Scaling** helps dynamically adjust resources.
- **Terraform** provides a declarative approach to defining Auto Scaling.
- **CloudWatch Alarms** trigger scaling based on defined thresholds.
- **SNS Notifications** enable monitoring and alerting.

