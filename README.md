# ros_trainning
## Tổng hợp các lệnh cơ bản
- Tạo workspace:

        mkdir -p ws/src

- Tạo package:

        catkin_create_pkg <package_name> <package_depend 1> <package_depend 2> ...

- Build các package có trong workspace:

        catkin_make

- Source cấu hình workspace:

        source <đường dẫn tới workspace>/devel/setup.bash

- Khởi động master:

        roscore

- Run node:

        rosrun <package_name> <executable_name>

  trong đó thì <executable_name> là tên file python (nếu node viết bằng python) hoặc tên được khai báo trong file CMakeList.txt add_executable() (nếu node viết bằng C++)
- Launch:

        roslaunch <package_name> <launch_file>.launch

- Các lệnh liên quan đến node:

        rosnode list                  # Liệt kê các node đang chạy
        rosnode info <node_name>      # Tra cứu thông tin <node_name>
        rosnode kill <node_name>      # Buộc <node_name> dừng hoạt động

- Các lệnh liên quan đến topic:

        rostopic list                   # Liệt kê các topic có trong hệ thống
        rostopic info <topic_name>      # Tra cứu thông tin <topic_name>
        rostopic hz <topic_name>        # Xem tần số dữ liệu được gửi lên <topic_name>
        rostopic echo <topic_name>      # Xem dữ liệu được gửi lên <topic_name>
        rostopic pub <topic_name> *tab tab  # Đẩy dữ liệu lên <topic_name> bằng dòng lệnh

- Các lệnh liên quan đến service:

        rosservice list                     # Liệt kê các service có trong hệ thống
        rosservice info <service_name>      # Tra cứu thông tin <service_name>
        rosservice call <service_name> *tab tab     # Gọi service server bằng dòng lệnh

## Tạo kiểu msg, srv, action tự định nghĩa
@ Lưu ý: khi tạo package cần depend vào message_generation và message_runtime
- Tạo thư mục /msg, /srv, /action
- Tương ứng tạo các file <MessageType>.msg, <ServiceType>.srv, <ActionType>.action
- Khai báo các trường thông tin trên các file
- Trong file CMakeList.txt, uncomment các dòng sau:

        ## Generate messages in the 'msg' folder
        add_message_files(
          FILES
          <MessageType>.msg
        )
        
        ## Generate services in the 'srv' folder
        add_service_files(
          FILES
          <ServiceType>.src
        )
        
        ## Generate actions in the 'action' folder
        add_action_files(
          FILES
          <ActionType>.action
        )
        
        ## Generate added messages and services with any dependencies listed here
        generate_messages(
          DEPENDENCIES
          geometry_msgs   nav_msgs   sensor_msgs   std_msgs   visualization_msgs
        )
        .
        .
        .
        catkin_package(
          INCLUDE_DIRS include
          CATKIN_DEPENDS geometry_msgs nav_msgs sensor_msgs std_msgs visualization_msgs message_runtime
        )

## Tạo node C++
- Tạo file header trong thư mục include, xem file mẫu trong /include/ros_template.h
- Tạo file cpp trong thư mục src, xem file mẫu trong /src/ros_template.cpp
- Uncomment dòng sau trong CMakeList.txt

        include_directories(
          include
          ${catkin_INCLUDE_DIRS}
        )

- Khai báo executable:

        add_executable(<executable_name> src/<source_code1>.cpp src/<source_code2>.cpp) # Một executable của C++ có thể có nhiều source code, nó sẽ chạy hàm main ở source_code1
        target_link_libraries(<executable_name> ${catkin_LIBRARIES})
        add_dependencies(<executable_name> ${catkin_EXPORTED_TARGETS})

- Mỗi khi sửa code C++ sẽ phải build lại

## Tạo node python
- Tạo file script python trong thư mục scripts, xem mẫu trong /scripts/ros_template.py
- Khai báo executable:

        catkin_install_python(PROGRAMS
          scripts/<source_code>.py
          DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
        )

- Mỗi khi sửa code python không cần build lại

## Topic API
- Topic hoạt động theo cơ chế Publish và Subscribe
- Các node publisher sẽ đẩy dữ liệu lên Topic
- Các node Subscriber mỗi khi có dữ liệu mới đẩy lên topic, nó sẽ thực thi hàm callback với đầu vào là dữ liệu vừa được gửi đến

![image](https://github.com/HoangfDungx/ros_trainning/assets/85065679/97a6e2f3-c0cd-47a3-a579-35b676af8ac2)

- Các bước tạo publisher/ subscriber
  B1: Khai báo kiểu topic

        #include <<package_name>/MessageType.h> // C++
        from <package_name>.msg import MessageType // python

  B2: Tạo publisher hoặc subscriber

        ros::Publisher pub;
        ros::Subscriber sub; // C++

  B3: Khai báo các publisher hoặc subscriber trong hàm khởi tạo

        pub = nh_.advertise<<package_name>::MessageType>(<topic_name>, 1);
        sub = nh_.subscribe(<topic_name>, 1, &ClassName::callbackFunction, this); // C++

        self.pub = rospy.Publisher(<topic_name>, MessageType, 1)
        self.sub = rospy.Subscriber(<topic_name>, MessageType, self.callback_function) // Python

  B4: Đối với publisher, dùng lệnh sau để gửi dữ liệu

        pub.publish(var); // C++
        self.pub.publish(var) // python

  B4: Đối với các subscriber, khai báo hàm callback như sau

        void callbackFunction(const <package_name>::MessageType::ConstPtr& msg) {} // C++ dùng biến msg như con trỏ
        def callback_function(self, msg)

## Service API
- Service hoạt động theo cơ chế Client và Server
- Client gửi một request đến Server và chờ Server trả kết quả
- Server khi nhận request sẽ thực hiện hàm callback và trả kết quả cho Client
- Có thể có nhiều Client nhưng chỉ có 1 Server

  ![image](https://github.com/HoangfDungx/ros_trainning/assets/85065679/79bc6291-de36-4efc-817a-708a47aaeead)

- Các bước tạo Service
  B1: Khai báo kiểu service

        #include <<package_name>/ServiceType.h> // C++
        from <package_name>.srv import ServiceType, ServiceResponse, ServiceRequest // python

  B2: Tạo các client hoặc server

        ros::ServiceServer server;
        ros::ServiceClient client; // C++
  
  B3: Khai báo các client hoặc server trong hàm khởi tạo

        client = nh_.serviceClient<<package_name>::ServiceType>(<service_name>);
        server = nh_.advertiseService(<service_name>, &ClassName::callbackFunction, this); C++

        self.client = rospy.ServiceProxy(<service_name>, ServiceType)
        self.server = rospy.ServiceServer(<service_name>, ServiceType, self.callback_function) // Python

  B4: Đối với client, dùng lệnh sau để gửi yêu cầu

        client.call(var); // C++, request gán trong var.request, response đọc từ var.response
        resp = self.server(var) // python

  B4: Đối với các server, khai báo hàm callback như sau

        bool callbackFunction(<package_name>::ServiceType::Request& req, <package_name>::ServiceType::Response& resp) {} // C++
        def callback_function(self, req) // python, phải return response (kiểu MessageTypeResponse) ở cuối

## Action API
- Cơ chế tương tự Service nhưng bản chất là Topic
- Client gửi goal đến server, client có thể không cần đợi server thực hiện xong
- Server thực hiện goal và đồng thời gửi feedback trong quá trình thực hiện
- Sau khi hoàn thành, server gửi lại result cho client

# TF
