# 用户新增和累计注册、开户人数统计

## Controller

```
@Authorization(needAuth = false)
    @RequestMapping(value = "/registerAndOpenAccountPeopleCount", method = RequestMethod.POST)
    public String registerAndOpenAccountPeopleCount(@Validated UserCountRequest userCountRequest, @CurrentLocale Locale locale)
            throws Exception {

//        Map resultMap = userSOAService.queryRegisterAndOpenAccountCount(userCountRequest.getTimeType(), userCountRequest
//                .getBeginTime(), userCountRequest.getEndTime());

        Map resultMap = new HashMap();

        SavedSearchDispatchArgs dispatchArgs = new SavedSearchDispatchArgs();
        dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"beginTime", userCountRequest.getBeginTime());
        dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"endTime", userCountRequest.getEndTime());

        //按日统计
        if("1".equals(userCountRequest.getTimeType())){
            dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"timeType", "\'%Y%m%d\'");
        }else if("2".equals(userCountRequest.getTimeType())){
            //按月统计
            dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"timeType", "\'%Y%m\'");
        }
        //注册
        dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"fieldName", "create_time");
        SplunkResult addedRegisterPeopleCountResult = splunkService.call("queryAddedPeopleCount", dispatchArgs);
        SplunkResult totalRegisterPeopleCountResult = splunkService.call("queryTotalPeopleCount", dispatchArgs);

        //开户
        dispatchArgs.add(SplunkConsts.ARGS_PRIFIX+"fieldName", "open_time");
        SplunkResult addedOpenAccountPeopleCountResult = splunkService.call("queryAddedPeopleCount", dispatchArgs);
        SplunkResult totalOpenAccountPeopleCountResult = splunkService.call("queryTotalPeopleCount", dispatchArgs);

        if(addedRegisterPeopleCountResult.isSuccess()&&addedOpenAccountPeopleCountResult.isSuccess()){
            resultMap.put("userAddedRegisterCountList",addedRegisterPeopleCountResult.getData());
            resultMap.put("userAddedOpenAccountCountList",addedOpenAccountPeopleCountResult.getData());
        }else {
            log.error("code:{},msg:{}",addedRegisterPeopleCountResult.getCode(),
                    addedRegisterPeopleCountResult.getMsg());
        }

        if(totalRegisterPeopleCountResult.isSuccess()&&totalOpenAccountPeopleCountResult.isSuccess()){
            resultMap.put("userTotalRegisterCountList",totalRegisterPeopleCountResult.getData());
            resultMap.put("userTotalOpenAccountCountList",totalOpenAccountPeopleCountResult.getData());
        }else {
            log.error("code:{},msg:{}",addedRegisterPeopleCountResult.getCode(),
                    addedRegisterPeopleCountResult.getMsg());
        }

        //查询无结果
        if(resultMap==null||resultMap.isEmpty()){
            return genErrorResult("110004",locale);
        }

        return this.genSuccessResult(resultMap,locale);
    }
```



