# unlinkParentAndChildren
## SDFS::UnlinkByParentUuidAndNames(parent_uuid, vector<string> children_names)

  For example, say there are 10 children in the child_map, and we are going to delete {c0, c3, c4, c7}
  Then it'll write four logs if we unlink it one by one. The logs will be like --
  ```
  <parent_uuid, stripe_id=-1, child_map={c1:uuid1, c2:uuid2, c3:uuid3, c4:uuid4, c5:uuid5, c6:uuid6, c7:uuid7, c8:uuid8, c9:uuid9}>.  # deleting c0
  <parent_uuid, stripe_id=-1, child_map={c1:uuid1, c2:uuid2, c4:uuid4, c5:uuid5, c6:uuid6, c7:uuid7, c8:uuid8, c9:uuid9}>.            # deleting c3
  <parent_uuid, stripe_id=-1, child_map={c1:uuid1, c2:uuid2, c5:uuid5, c6:uuid6, c7:uuid7, c8:uuid8, c9:uuid9}>.                      # deleting c4
  <parent_uuid, stripe_id=-1, child_map={c1:uuid1, c2:uuid2, c5:uuid5, c6:uuid6, c8:uuid8, c9:uuid9}>.                                # deleting c7
  ```
    
    
  ### Basic Idea
    Divide the total operation into chunks, so the size of logs for each chunk would be smaller than 128MB. 
    - Calculate the size of the logs that would be generated for unlinking these children to the parent. For example, the size of current metadata entry is 100b, unlinking one child will result in a shorter child_map and the size of metadata will be 98b. Unlinking one more child will lead the size of metadata entry to be 96b...Most like a arithmetic sequence. 
    - If the size is smaller than 128MB, we can just use existing approach to unlink them one by one and expect no backpressure error.
    - If the size is bigger than 128MB, divide all operations into chunks. The chunk size/number should be dynamic. 
      -- To decide the number of chunks and size of each chunk, we do as follow:
         --- Calculate the total size of the logs `S (MB)` if we do unlink one by one, just as the example shows above
         --- chunk_size = ceil(S/128). `chunk_size` refers to how many children we want to unlink in a single operation. To be more specific, if we unlink one child each time as our current approach does, the chunk_size will be 1.
         --- All operations in a single chunk should be in one transaction: all happen or none happens
     - Maybe introduce multi-thread so we can do parrallel unlinking for chunks
        
    
         
         
