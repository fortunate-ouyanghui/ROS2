# API
## 服务通信
```C
服务端：
创建服务端对象并注册回调函数：
server_=this->create_service<interface::srv::AddInts>("/srv/AddInts",bind(&Server::add,this,_1,_2),10);
当客户端提交业务需求时，服务端会自动调用add函数，其中_1和_2代表请求对象和反馈对象
void add(const interface::srv::AddInts::Request::SharedPtr req,const interface::srv::AddInts::Response::SharedPtr res) const
{
    res->sum=req->num1+req->num2;
    RCLCPP_INFO(this->get_logger(),"请求数据:(%d,%d) 响应结果(%d)",req->num1,req->num2,res->sum);
}


客户端：
连接服务端
client_->wait_for_service(1s)

发送请求
client_->async_send_request(req);

获取服务端的处理结果
clcpp::spin_until_future_complete(client,res)==rclcpp::FutureReturnCode::SUCCESS
```

## 动作通信
```C
服务端：
3个回调函数（在创建动作服务端对象时绑定，事件发生时，自动触发调用）：
判断客户端提交的业务是否合法：
using GoalCallback = std::function<GoalResponse(
        const GoalUUID &,
        std::shared_ptr<const typename ActionT::Goal>)>;

接收到客户端取消任务的请求，需要处理：
using CancelCallback = std::function<CancelResponse(std::shared_ptr<ServerGoalHandle<ActionT>>)>;

生成连续反馈
using AcceptedCallback = std::function<void (std::shared_ptr<ServerGoalHandle<ActionT>>)>;



判断任务是否取消
goal_handle->is_canceling()
当任务被取消时，返回最终结果用这个函数
goal_handle->canceled(result);
生成连续反馈
goal_handle->publish_feedback(feedback);
业务完成后，发布最终结果用这个函数
goal_handle->succeed(result);



客户端：
连接服务端
action_client_->wait_for_action_server(10s)
发送请求
action_client_->async_send_goal(goal,options);//options要注册三个函数（返回反馈数据；返回服务端是否同意取消任务；返回任务执行的状态，如果任务完成，则回调函数的参数会包含结果在内）
```
