.. This is an included how-to. 

The following example shows how to reload the configuration of a |chef client| using the |resource remote_file| resource to:

* using an ``if`` statement to check whether the plugins on a node are the latest versions
* identify the location from which |ohai| plugins are stored
* using the ``notifies`` attribute and a |resource ruby block| resource to trigger an update (if required) and to then reload the |client rb| file.

.. code-block:: ruby

   directory node[:ohai][:plugin_path] do
     owner "chef"
     recursive true
   end
   
   ruby_block "reload_config" do
     block do
       Chef::Config.from_file("/etc/chef/client.rb")
     end
     action :nothing
   end
   
   if node[:ohai].key?(:plugins)
     node[:ohai][:plugins].each do |plugin|
       remote_file node[:ohai][:plugin_path] +"/#{plugin}" do
         source plugin
         owner "chef"
         notifies :create, resources(:ruby_block => "reload_config")
       end
     end
   end
