## 说明

上一次更新还是2年前，这段时间一直想不到好的解决方案来完成心目中的样子，所以仓库一直处于停滞的状态，而且流产了 2.0版本。这次完全重构了改项目，前端采用 React 框架。

> 注意：只是预览版，没有开放仓库。代码可能会有变更，尽请期待。如果您感兴趣请加群了解！![群号：649722075](http://orsbhqabt.bkt.clouddn.com/e_group.png)

### 前端

前端采用 React + React-Route。

### 效果图：

![React](http://orsbhqabt.bkt.clouddn.com/react-5.gif)
![React](http://orsbhqabt.bkt.clouddn.com/react-2.gif)
![React](http://orsbhqabt.bkt.clouddn.com/react-3.gif)
![React](http://orsbhqabt.bkt.clouddn.com/react-4.gif)

### 后端

后端采用 Laravel 5.5 版本。并且全部用 composer 把 各个服务打包成包。

### 效果图：

![Composer 包](http://orsbhqabt.bkt.clouddn.com/1.png)

### 代码示例（用 GIF 里面的 管理员这个模块举例）：

#### 列表页 Table Dom 描述

```
    /**
     * 获得 table 的描述
     *
     * @return Table
     */
    public function getTable(): Table
    {
        return (new Table(['rowTitle' => 'user_name']))
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => 'ID',
                    'dataIndex' => 'id',
                    'width'     => 50,
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'         => '用户名',
                    'dataIndex'     => 'user_name',
                    'width'         => 150,
                    'editableWidth' => 250,
                ])
                    ->setEditableFormItem(function (Input $input) {
                        return $input->setName('user_name')->setRules(function (Rule $rule) {
                            return $rule->setMessage('用户名不能为空！')->setRequired(true);
                        });
                    });
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'         => '角色',
                    'dataIndex'     => 'role_name',
                    'width'         => 150,
                    'editableWidth' => 350,
                ])
                    ->setEditableFormItem(function (TreeSelect $treeSelect) {
                        return $treeSelect
                            ->setName('role_id')
                            ->setShowCheckedStrategy(FormItemConst::SHOW_ALL)
                            ->setTreeData(UtilityLibrary::transColums((new Roles())->recursiveData(), [
                                'id'       => 'value',
                                'name'     => 'label',
                                'children' => 'children',
                            ]))
                            ->setRules(function (Rule $rule) {
                                return $rule->setMessage('父级角色不能为空！')->setRequired(true)->setType('number');
                            });
                    })
                    ->setEditableColunmAlias('role_name')
                    ->setEditableType(TableConst::EDITABLE_POPOVER);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '头像',
                    'dataIndex' => 'avatar_str',
                    'type'      => 'image',
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '手机',
                    'dataIndex' => 'mobile',
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '邮箱',
                    'dataIndex' => 'email',
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '状态',
                    'dataIndex' => 'status_value',
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '登录次数',
                    'dataIndex' => 'login_number',
                    'sorter'    => true,
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => 'IP',
                    'dataIndex' => 'last_login_ip',
                ]);
            })
            ->setColumns(function (Column $column) {
                return $column->add([
                    'title'     => '登录时间',
                    'dataIndex' => 'updated_at',
                ]);
            });
    }
```

#### 列表 搜索框 Dom 描述

```
    /**
     * 获得 列表搜索的表单 的描述
     *
     * @return SearchForm
     */
    public function getSearchForm(): SearchForm
    {
        return $this->getEditForm()
            ->only('user_name', 'email', 'mobile', 'status', 'role_id')
            ->update('status', function (RadioGroup $radioGroup) {
                return $radioGroup->setHasAll()->setDefaultValue(null);
            })
            ->update('role_id', function (TreeSelect $treeSelect) {
                return $treeSelect->setHasAll([
                    'value' => 0,
                    'label' => '全部'
                ])->setDefaultValue(0);
            })
            ->map(function (FormItem $formItem) {
                return $formItem->toDefaultSearchItem()->removeRules();
            })
            ->toSearchForm();
    }
```

> 我们可以复用一个组件，然后过滤掉里面不需要的表单 Item，或者设置某个表单 Item，甚至可以批量操作全部 Item。

#### 编辑页面表单 Dom 描述

```
/**
     * 获得 编辑的表单 的描述
     *
     * @return Form
     */
    public function getEditForm(): Form
    {
        return Form::create()
            ->setItem(function (Input $input) {
                return $input->setName('user_name')->setLabel('用户名')->setCol(function (Col $col) {
                    return $col->setSpan(12)->setOffset(12)->setPull(12);
                });
            })
            ->setItem(function (TreeSelect $treeSelect) {
                return $treeSelect
                    ->setLabel('父级角色')
                    ->setName('role_id')
                    ->setShowCheckedStrategy(FormItemConst::SHOW_ALL)
                    ->setTreeData(UtilityLibrary::transColums((new Roles())->recursiveData(), [
                        'id'       => 'value',
                        'name'     => 'label',
                        'children' => 'children',
                    ]))
                    ->setCol(function (Col $col) {
                        return $col->setSpan(12)->setOffset(12)->setPull(12);
                    })
                    ->setRules(function (Rule $rule) {
                        return $rule->setMessage('父级角色不能为空！')->setRequired(true)->setType('number');
                    });
            })
            ->setItem(function (Password $password) {
                return $password->setName('password')->setLabel('密码')->setCol(function (Col $col) {
                    return $col->setSpan(12);
                })->setRules(function (Rule $rule) {
                    return $rule->setValidator('validatePassword');
                });
            })
            ->setItem(function (Password $password) {
                return $password->setName('rpassword')->setLabel('确认密码')->setCol(function (Col $col) {
                    return $col->setSpan(12);
                })->setRules(function (Rule $rule) {
                    return $rule->setValidator('validateRPassword');
                });
            })
            ->setItem(function (Input $input) {
                return $input->setName('email')->setLabel('邮箱')->setCol(function (Col $col) {
                    return $col->setSpan(12);
                })->setRules(function (Rule $rule) {
                    return $rule->setMessage('邮箱不能为空！')->setRequired(true);
                })->setRules(function (Rule $rule) {
                    return $rule->setMessage('邮箱格式不正确！')->setType('email');
                });
            })
            ->setItem(function (Input $input) {
                return $input->setName('mobile')->setLabel('手机')->setCol(function (Col $col) {
                    return $col->setSpan(12);
                })->setRules(function (Rule $rule) {
                    return $rule->setMessage('手机号码格式不正确！')->setType('mobile')->setRequired(true);
                });
            })
            ->setItem(function (AvatarUpload $avatarUpload) {
                return $avatarUpload
                    ->setName('avatar')
                    ->setLabel('头像')
                    ->setRouteNameAction('admin/upload')
                    ->setCol(function (Col $col) {
                        return $col->setSpan(12);
                    });
            })
            ->setItem(function (RadioGroup $radioGroup) {
                return $radioGroup
                    ->setName('status')
                    ->setLabel('状态')
                    ->setDefaultValue(Admin::STATUS_1)
                    ->setOptions($this->getModel()->statusArrayToOptions())
                    ->setCol(function (Col $col) {
                        return $col->setSpan(12)->setOffset(12)->setPull(12);
                    });
            });
    }
    ```
## 期待这次能够满足您的需求。



