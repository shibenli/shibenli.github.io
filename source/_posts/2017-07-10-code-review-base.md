---
title: 代码Review系列之2-基类分布
date: 2017-07-10 10:42:58
categories: Android
tags:
    - Android
    - 代码Review
---
## 基类分布

基类分布这里主要分两个方面来说明，包括：Activity的基类（Fragment的基类是类似的）、功能的基类。

### Activity的基类
![基类分布](/images/2017/基类封装.png)

借助于[butterknife](https://github.com/JakeWharton/butterknife "butterknife")的控件、事件的注入功能，封装了获取layout的方法和一些初始化的新的生命周期方法。

<!-- more --> 

### 功能的基类
![基于功能的基类分布](/images/2017/基类封装-删除、记录、问题反馈.png)

首先，上面这幅图是从详情页面的基类冲抽出来的，基于此基类提供删除、充值记录、还款记录、问题反馈的基本功能。

由于多个页面都有上述的菜单模块，这里提供基于基类的事件监听机制。通过具体界面的菜单项的配置、监听器的配置，来达到事件处理重用、具体的处理方式方法又提供了可扩展（重写deleteBill方法实现）。


## 附录：

完整的代码如下

```

public abstract class BaseWithDeleteActivity extends AbsBaseActivity {

    /**
     * 账单信息
     */
    protected BillInfo billInfo;

    /**
     * 条目监听
     */
    protected View.OnClickListener itemListener = new View.OnClickListener() {
        PopWindowHelper.PopDataInfo popDataInfo;

        @Override
        public void onClick(View v) {
            Object tag = v.getTag(R.id.cb_item_tag);
            if (tag != null && tag instanceof PopWindowHelper.PopDataInfo) {
                popDataInfo = (PopWindowHelper.PopDataInfo) tag;
                Intent intent = null;
                switch (popDataInfo.getTitleid()) {
                    case R.string.tab_bill_detail_delete_card: { //删除卡片
                        TooltipUtils.showDialog(mActivity, "确定删除？", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                deleteBill(billInfo);
                            }
                        }, null, "确定", "取消");
                    }
                    break;
                    case R.string.tab_bill_detail_pay_record://充值记录
                        intent = new Intent(mActivity, LifeHistoryActivity.class);
                        intent.putExtra(LifeHistoryActivity.KEY_TITLE, getString(popDataInfo.getTitleid()));
                        intent.putExtra(LifeHistoryActivity.KEY_TYPE, "shenghuojiaofei");
                        break;
                    case R.string.tab_bill_detail_record: {//还款记录
                        intent = new Intent(mActivity, PaymentHistoryActivity.class);
                        intent.putExtra(PaymentHistoryActivity.KEY_TITLE, getString(popDataInfo.getTitleid()));
                    }
                    break;
                    case R.string.tab_bill_detail_feed_back: {//问题反馈
                        intent = new Intent(mActivity, FeedbackActivity.class);
                    }

                    break;
                }
                if (intent != null) {
                    startActivity(intent);
                }

            }
        }
    };

    /**
     * 删除账单的方法
     *
     * @param info 账单信息
     */
    protected void deleteBill(final BillInfo info) {
        Map<String, Object> params = new HashMap<>();
        params.put("orderId", info.getOrderId());
        params.put("orderType", info.getOrderType());
        httpUtil.doPostByJson(HttpUtil.getServiceUrl(InterfaceConfig.bill.DELETE_BILLS), params, deleteBillListener);
    }

    /**
     * 删除订单接口回调
     */
    protected RequestListener deleteBillListener = new AbsRequestListener(this) {
        @Override
        public void onSuccess(String result) {
            ResultInfo resultInfo = JsonUtils.json2Object(result, ResultInfo.class);
            if (resultInfo.isOperationResult()) {
                finish();
            }

            if (null != resultInfo) {
                TooltipUtils.showToastL(mActivity, resultInfo.getDisplayInfo(), 100);
            }
        }
    };

}

```