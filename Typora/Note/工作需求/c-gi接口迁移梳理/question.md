#### **1. /sz-c-gi/medical/query/orderDetail逻辑梳理**

1. 根据订单编号，获取用户订单信息

   orderService.queryOrderByOrderCode(reqDto.getOrderCode())

   orderMapper.getByExampleWithPage(OrderDO)

   返回OrderDO的List

   

2. 根据订单编号，获取问诊详情内容

   questionOrderService.getQuestionOrderDosMap(orderCodes);

   

3. 根据处理流水号，获取支付信息

   dealService.getDealInfoListByDealCode(orderDo.getDealCode());

   

4. 如果是新的问诊订单，获取问诊订单的关联医生订单相关信息并设置

   - 新的问诊订单只有ios有吗

   - medicalService.medicalOrderDetailQuery(request)

   不是的话，根据前面获取的orderDO设置订单状态

   

5. 将以上获取信息封装至context上的map



#### **2. /sz-c-gi/medical/query/dialogue逻辑梳理**

1. 根据订单编号，获取用户订单信息

   orderService.queryOrderByOrderCode(reqDto.getOrderCode())

   orderMapper.getByExampleWithPage(OrderDO)

   返回OrderDO的List

2. 如果OrderDO的订单来源APP是62（来自app）

   1. 如果支付状态是1（0初始，1支付中，2支付成功，3支付失败）

      1. 根据订单编号查找问诊订单

         questionOrderMapper.getByOrderCode(orderCode)

      2. 创建用户对话记录（包括文字、图片url、语音url）
      3. 设置订单状态为11（未支付）

   2. 支付状态是其他时

      1. 封装请求体（orderCode、StartId、ServiceCode）

      2. 调用MedicalService获取对话记录

         1. 调用IOrderDetailService根据请求中businessOrderCode获取t_order_detail订单详情

         2. 根据t_order_detail中orderCode获取t_order信息

         3. 调用IOrderDistributionRecordService中queryByOrderCode从t_order_distribution_record获取订单分配记录，并且sz_status不是5，8

            sz状态(-1待分配，1待解读，2审核中3审核通过，4审核不通过5重新分配，6进行中，7关闭,8修复)

         4. 根据3中获取的分配id从t_order_dialogue_record中获取对话记录（orderDialogueRecordMapper）

         5. 根据orderDetailDo中的**隧道数据json**中的templeteCode通过IReportAssessTempleteService从t_report_assess_title、t_report_templete_title_conn、t_report_assess_templete三个表中获取问题列表数据

         6. 将5中获取的问题列表根据TitleCode和QuestionName封装成map

         7. 问题记录不为空，根据4中获得的orderDialogueRecordDos获取医生的记录getDialogueContent转成ReportAssessTitleDto报告题目评估对象。接着转换问题对象，将选择项塞到对象中

            1. 根据templeteCode调用reportAssessTempleteMapper从下表中查询问题选项

            ```
            asgard_legend.t_report_templete_title_conn rat,
            asgard_legend.t_report_assess_title ratt, asgard_legend.t_report_assess_option rao
            ```

         8. 将选中的选项设置为已选择放入dto中

         9. 将对话记录封装到返回体中

         10. 查询分配医生头像

         11. 补充问题交互次数，查询redis中缓存的个数，如果redis中交互个数与数据库不一致，则查询数据库。

      3. medicalService返回成功，设置对话列表，订单状态，备注，医生头像，否则查询失败抛出异常

   3. 订单来源不是62（app）

      1. 根据订单编号获取问诊订单

      ​	questionOrderMapper.getByOrderCode(orderCode)

      2. 根据问诊订单创建时间创建用户对话记录dto，dto解码并设置对话内容
      3. 当encoded为空时，可能以前存在于content字段，dto设置对话内容
      4. 从1得到的数据  获取问题状态（0-未解答，1-已分配未解答 -1 解答失败 2-已解答）以及对应文案
      5. 判断分配id不为空，并且问题已解决
         1. 根据分配id从T66_QUESTION_ORDER_ASSIGN获取回答内容，解密并设置内容
         2. 设置语音解答信息
         3. 查询获取医生头像

   4. 封装回答记录、订单状态、备注、医生头像



#### 3. /sz-c-gi/medical/query/orderList逻辑梳理

1. 调用memberService根据用户手机号码从T66_MEMBER_INFO获取会员信息（首先通过线程获取用户登录信息，获取用户手机号码）
2. 封装orderDO（会员code和订单类型）用于后续查询订单信息
3. 调用orderService根据2的orderDo和分页信息从T66_ORDER LEFT JOIN T66_QUESTION_ORDER 获取订单列表数据信息
4. 从3中获取的订单列表封装成订单编号列表，从T66_QUESTION_ORDER获取问诊订单列表
5. 根据获取的信息封装返回参数



#### 4. /sz-c-gi/medical/query/getMedicalDoctorList逻辑梳理

1. 调用memberService根据用户手机号码从T66_MEMBER_INFO获取会员信息（首先通过线程获取用户登录信息，获取用户手机号码）
2. 调用orderService根据会员code从T66_ORDER查询当天免费问诊订单
3. 调用serviceInfoService根据服务Code从T66_SZ_SERVICE_INFO获取服务信息
4. 根据上述获取的信息封装返回参数



#### 5. /sz-c-gi/medical/query/consultantUserOrderList逻辑梳理

1. 获取用户登录信息
2. 根据用户idSet 从T66_QUESTION_ORDER LEFT JOIN T66_ORDER 获取用户的订单总数
3. 设置分页信息
4. 根据用户idSet和分页信息从 T66_QUESTION_ORDER LEFT JOIN T66_ORDER 获取用户所有订单详情
5. 根据上述获取的信息封装返回参数

