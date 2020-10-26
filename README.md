# -
python代码
main


from Login_Form import LoginForm
from Index_Form import IndexForm
from UserInfo_Form import UserInfoForm
from ShowChart_Form import ShowChartForm
from Table import TableForm,TableForm_User
from Store_analyz_form import Store_analyz_Form
from Compound_schedule_Form import Compound_scheduleForm
from WareInfo_Dialog import WareInfoDialog
from UserManage_Dialog import UserManageDialog
from custom_dialog import Custom_Dialog
from PyQt5.Qt import *
from ConnectDatabase import OperateDatabase
from Store_out_form import Store_out_Form
from systemInfo_form import SystemInfoForm
import sys
from MyDialog import MyDialog
import sip
import traceback
import qdarkgraystyle
# import PyQt5_stylesheets

# 登录的身份
ISADMIN = 0

# 登录的ID
person = "P100000"

if __name__ == '__main__':
    app = QApplication(sys.argv)
    app.setWindowIcon(QIcon(":/window/images/window_icon.png"))
    print("***********************************")
    login_form = LoginForm()
    login_form.setWindowFlags(Qt.WindowStaysOnTopHint)
    index_form = IndexForm()
    index_form.setStyleSheet(qdarkgraystyle.load_stylesheet_pyqt5())
    layout = QVBoxLayout()

    # 清空界面
    def removewidget():
        for i in range(layout.count()):
            sip.delete(layout.itemAt(i).widget())

    # 槽函数
    def login_check_btn(account, password):
        try:
            accdb = OperateDatabase("./resource/database/SystemDatabase.db", "user")
            value_list = accdb.selectData_onerow(account)
            if value_list:
                if password == value_list[2]:
                    if value_list[3] == 1:
                        global ISADMIN
                        ISADMIN = 1
                        index_form.setstatusbar ('人员身份：管理员')
                    else:
                        index_form.setstatusbar ('人员身份：员工')
                    index_form.show()
                    global person
                    person = account
                    login_form.close()
                    return
                else:
                    login_form.show_error_animation()
            else:
                messagebox = MyDialog.show_dialog("消息提示", "该账户不存在！", ":/messagebox/images/error.png", login_form)
                messagebox.show()
                def check(btn):
                    role = messagebox.buttonRole(btn)
                    if role == QMessageBox.YesRole:
                        login_form.clear_content()
                messagebox.buttonClicked.connect(check)
        except Exception:
            exc_info = traceback.format_exc ()
            print (exc_info)

    def toggle_form(widget, item_text):
        if item_text == "仓库信息":
            serachCondition_dic = {'按仓库名称查询': '仓库名称', '按料箱编码查询': 'ID', '按货物种类查询': '货物种类', '按库位代码查询': '库位代码'}
            table = TableForm(serachCondition_dic, 20, "./resource/database/SystemDatabase.db", "仓库信息表")
            table.setTabelHead(["料箱编码", "仓库名称", "货品编号", "货品总件", "库位代码"])
            if ISADMIN == index_form.MANAGER:
                table.addcolunm()
            def wareInfo(data_list, accdb):
                wareinfo = WareInfoDialog(table)
                wareinfo.show_wareInfo (data_list, accdb)
                def success():
                    table.searchButtonClicked ()
                wareinfo.alter_sure_signal.connect (success)
                wareinfo.open ()
            table.alter_data_signal.connect (wareInfo)
            removewidget()
            layout.addWidget(table)
            widget.setLayout(layout)
            return

        if item_text == "用户信息":
            if ISADMIN == index_form.USER:
                userinfo = UserInfoForm()
                da = OperateDatabase("./resource/database/SystemDatabase.db", "员工信息表")
                userinfo.setShowInfo(da.selectData_onerow(person))
                removewidget()
                layout.addWidget(userinfo)
                widget.setLayout (layout)
                return
            if ISADMIN == index_form.MANAGER:
                serachCondition_dic = {'按员工编码查询': 'user.ID', '按员工姓名查询': '姓名', '按员工任职部门查询': '任职部门', '按员工职务查询': '职务',
                                       '按员工性别查询': '性别', }
                table = TableForm_User(serachCondition_dic, 20, "./resource/database/SystemDatabase.db", "货物入库表")
                table.setTabelHead(["员工编号","姓名","登录密码","性别","出生日期","邮箱","办公电话","任职部门","职务"])
                table.addcolunm()
                def userInfo(data_list, accdb):
                    userinfo = UserManageDialog(table)
                    userinfo.show_userInfo (accdb.select_userInfo(data_list[0]), accdb)
                    def success():
                        table.searchButtonClicked()
                    userinfo.alter_sure_signal.connect(success)
                    userinfo.open()
                table.alter_data_signal.connect(userInfo)
                removewidget()
                layout.addWidget(table)
                widget.setLayout(layout)
                return

        if item_text == "按月统计":
            showchart_form = ShowChartForm()
            removewidget ()
            layout.addWidget(showchart_form)
            widget.setLayout (layout)
            return

        if item_text == "入库信息查询":
            serachCondition_dic = {'按货物名称查询': '货物名称', '按入库编码查询': 'ID', '按仓库名称查询': '仓库名称', '按入库时间查询': '入库日期','按货物规格查询': '货物规格'}
            table = TableForm(serachCondition_dic, 50, "./resource/database/SystemDatabase.db", "货物入库表")
            table.setTabelHead(["入库编码", "货物编号", "货物名称", "仓库名称", "货物规格", "入库数量","入库时间"])
            removewidget()
            layout.addWidget(table)
            widget.setLayout(layout)
            return

        if item_text == "出库信息查询":
            serachCondition_dic = {'按货物名称查询': '货物名称', '按出库编码查询': 'ID', '按仓库名称查询': '仓库名称', '按出库时间查询': '出库日期',
                                   '按货物规格查询': '货物规格', "按出库数量查询" : "出库数量"}
            table = TableForm (serachCondition_dic, 50, "./resource/database/SystemDatabase.db", "货物出库表")
            table.setTabelHead (["出库编码", "仓库名称", "货物名称", "货物规格", "出库数量", "出库时间"])
            removewidget ()
            layout.addWidget (table)
            widget.setLayout (layout)
            return

        if item_text == "库存信息查询":
            serachCondition_dic = {'按货物名称查询': '货物名称', '按货物编码查询': 'ID', '按仓库编码查询': '仓库编码', '按物流件数': '物流件数',
                                   '按盘存件数': '盘存件数'}
            table = TableForm (serachCondition_dic, 50, "./resource/database/SystemDatabase.db", "在库盘存表")
            table.setTabelHead (["货物编码", "货物名称", "仓库编码", "物流件数", "盘存件数"])
            removewidget ()
            layout.addWidget (table)
            widget.setLayout (layout)
            return

        if item_text == "运行策略及时间计算":
            compound_scheduleform = Compound_scheduleForm ()
            removewidget ()
            layout.addWidget(compound_scheduleform)
            widget.setLayout (layout)
            return



        if item_text == "仓储参数设置":
            system_info_form = SystemInfoForm()
            if ISADMIN == index_form.USER:
                system_info_form.hidden_btn()
            removewidget()
            layout.addWidget(system_info_form)
            widget.setLayout (layout)
            return

        if item_text == "库存分析":
            store_analyz_form = Store_analyz_Form()
            removewidget()
            layout.addWidget(store_analyz_form)
            widget.setLayout (layout)
            return

    def menu_Triggered(action):
        if action.text() == "重新登录":
            try:
                index_form.close()
                login_form.show()
                login_form.clear_content()
                removewidget()
                index_form.cannot_bkg(False)
                # index_form.show_item()
                return
            except Exception:
                exc_info = traceback.format_exc ()
                print(exc_info)
        if action.text() == "退出":
            app.quit()
            return

    # 信号连接
    login_form.check_btn_signal.connect(login_check_btn)
    index_form.show_form_signal.connect(toggle_form)
    index_form.menu_triggered_signal.connect(menu_Triggered)

    login_form.show()
    sys.exit(app.exec_())
