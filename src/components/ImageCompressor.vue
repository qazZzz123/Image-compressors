<template>
  <div class="container">
    <h1 class="title">智能图片压缩工具</h1>
    <div class="upload-container">
      <div id="drop-zone" :class="{
        'dragover': isDragging,
        'disabled': isUploadDisabled
      }" @dragenter.prevent="handleDragEnter" @dragleave.prevent="handleDragLeave" @dragover.prevent
        @drop.prevent="handleDrop" @click="triggerFileInput">
        <div class="upload-icon" :class="{ 'uploading': isUploading }">
          <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24">
            <path fill="currentColor"
              d="M19.35 10.04A7.49 7.49 0 0 0 12 4C9.11 4 6.6 5.64 5.35 8.04A5.994 5.994 0 0 0 0 14c0 3.31 2.69 6 6 6h13c2.76 0 5-2.24 5-5c0-2.64-2.05-4.78-4.65-4.96zM14 13v4h-4v-4H7l5-5l5 5h-3z" />
          </svg>
        </div>
        <p class="upload-text">{{ uploadText }}</p>
        <span class="upload-tip">支持 JPG、PNG、WebP 等格式，单个文件最大 20MB</span>
        <input ref="fileInput" type="file" accept="image/*" multiple class="file-input" @change="handleFileSelect">
        <div class="upload-counter">{{ images.length }}/{{ MAX_UPLOAD_COUNT }}</div>
      </div>
    </div>

    <div id="preview-container" v-if="images.length > 0">
      <div class="compression-settings">
        <div class="batch-operations">
          <div class="left-actions">
            <div class="checkbox-wrapper">
              <label class="select-all-label">
                <input type="checkbox" v-model="isAllSelected" @change="toggleSelectAll" class="select-all-checkbox">
                <span class="checkbox-text">全选</span>
              </label>
              <span v-if="selectedCount > 0" class="selected-count">
                已选择 <span class="count-number">{{ selectedCount }}</span> 项
              </span>
            </div>
          </div>
          <div class="right-actions">
            <el-button plain type="primary" :disabled="!hasSelected" @click="batchDownload" class="ant-btn">
              <svg class="anticon" viewBox="0 0 1024 1024" width="1em" height="1em" fill="currentColor">
                <path d="M624 706.3h-74.1V464c0-4.4-3.6-8-8-8h-60c-4.4 0-8 3.6-8 8v242.3H400c-6.7 0-10.4 7.7-6.3 12.9l112 141.7a8 8 0 0 0 12.6 0l112-141.7c4.1-5.2.4-12.9-6.3-12.9z"/>
                <path d="M811.4 366.7C765.6 245.9 648.9 160 512.2 160S258.8 245.8 213 366.6C127.3 389.1 64 467.2 64 560c0 110.5 89.5 200 200 200h88v-64h-88c-74.8 0-136-61.2-136-136 0-71.6 57.9-130.1 129.1-131.9l30.3-.7 14.6-27.2c37.3-69.4 107.6-112.2 183.9-112.2 77.3 0 147.5 42.6 184.9 112.1l14.6 27.2 30.3.7c71.2 1.8 129.1 60.3 129.1 131.9 0 74.8-61.2 136-136 136h-88v64h88c110.5 0 200-89.5 200-200 0-92.7-63.3-170.7-149-193.2z"/>
              </svg>
              <span>批量下载</span>
            </el-button>
            <el-button plain type="danger" :disabled="!hasSelected" @click="batchDelete" class="ant-btn">
              <svg class="anticon" viewBox="0 0 1024 1024" width="1em" height="1em" fill="currentColor">
                <path d="M832 256h-184v-72c0-30.9-25.1-56-56-56h-160c-30.9 0-56 25.1-56 56v72h-184c-17.7 0-32 14.3-32 32v32c0 4.4 3.6 8 8 8h56v544c0 30.9 25.1 56 56 56h480c30.9 0 56-25.1 56-56V328h56c4.4 0 8-3.6 8-8v-32c0-17.7-14.3-32-32-32zM432 184c0-4.4 3.6-8 8-8h144c4.4 0 8 3.6 8 8v72h-160v-72zm288 656c0 4.4-3.6 8-8 8H312c-4.4 0-8-3.6-8-8V328h416v512z"/>
                <path d="M448 416c-8.8 0-16 7.2-16 16v288c0 8.8 7.2 16 16 16s16-7.2 16-16V432c0-8.8-7.2-16-16-16zM576 416c-8.8 0-16 7.2-16 16v288c0 8.8 7.2 16 16 16s16-7.2 16-16V432c0-8.8-7.2-16-16-16z"/>
              </svg>
              <span>批量删除</span>
            </el-button>
          </div>
        </div>
        <div class="quality-control">
          <div class="quality-header">
            <span class="quality-title">默认压缩质量</span>
            <span class="quality-tip">调节压缩率可以平衡图片质量和文件大小</span>
          </div>
          <div class="slider-container">
            <input type="range" v-model="defaultQuality" min="0" max="100" class="slider" @input="updateAllEstimates">
            <span class="quality-value">
              <span class="number">{{ defaultQuality }}</span><span class="percent-sign">%</span>
            </span>
          </div>
        </div>
        <div class="total-stats">
          <div class="stat-item">
            <span>原始总大小</span>
            <strong>{{ formatFileSize(totalOriginalSize) }}</strong>
          </div>
          <div class="arrow">→</div>
          <div class="stat-item">
            <span>预计压缩后</span>
            <strong>{{ formatFileSize(totalEstimatedSize) }}</strong>
          </div>
          <div class="saving-rate" :style="{ '--saving-color': savingRateColor }">
            节省 {{ savingRate }}%
          </div>
        </div>
      </div>

      <div class="images-container">
        <TransitionGroup name="image-list" tag="div" class="image-list">
          <div v-for="(image, index) in paginatedImages" :key="image.file.name + index" class="image-item" :class="{
            'processing': image.processing,
            'selected': image.selected
          }">
            <input type="checkbox" v-model="image.selected" @change="updateSelectAll" class="image-checkbox">
            <button class="delete-btn" @click.stop="deleteImage(index)" :disabled="image.processing">
              <span class="delete-icon">×</span>
            </button>
            <div class="image-preview">
              <img :src="image.preview" alt="预览图">
              <div class="image-overlay">
                <div class="image-dimensions">{{ image.dimensions }}</div>
              </div>
            </div>
            <div class="image-info">
              <div class="file-name">{{ image.file.name }}</div>
              <div class="individual-quality-control" v-if="showQualityControl(image.outputFormat)">
                <label>压缩质量</label>
                <div class="slider-container">
                  <input type="range" v-model="image.quality" min="0" max="100" class="slider"
                    @input="updateEstimate(getOriginalIndex(index))">
                  <span class="quality-value">{{ image.quality }}%</span>
                </div>
              </div>
              <div class="format-note" v-else>
                {{ getFormatNote(image.outputFormat) }}
              </div>
              <div class="size-info">
                <div class="size-item">
                  <span>原始大小</span>
                  <strong>{{ formatFileSize(image.originalSize) }}</strong>
                </div>
                <div class="size-arrow"></div>
                <div class="size-item">
                  <span>预计大小</span>
                  <strong v-if="image.estimating" :style="{ color: '#00bcd4' }">计算中...</strong>
                  <strong v-else>{{ formatFileSize(image.estimatedSize) }}</strong>
                </div>
              </div>
              <div class="compression-progress" v-if="image.processing">
                <div class="progress-bar" :style="{ width: image.progress + '%' }"></div>
              </div>
              <button @click="downloadImage(index)" class="download-btn" :disabled="image.processing">
                <span>{{ image.processing ? '压缩中...' : '下载' }}</span>
                <svg v-if="!image.processing" xmlns="http://www.w3.org/2000/svg" width="16" height="16"
                  viewBox="0 0 24 24">
                  <path fill="currentColor" d="M19 9h-4V3H9v6H5l7 7l7-7zM5 18v2h14v-2H5z" />
                </svg>
                <div v-else class="spinner"></div>
              </button>
              <div class="format-control">
                <label>转换格式</label>
                <select v-model="image.outputFormat" @change="handleFormatChange(getOriginalIndex(index))">
                  <option value="image/jpeg">JPG</option>
                  <option value="image/png">PNG</option>
                  <option value="image/webp">WebP</option>
                  <option value="image/avif">AVIF</option>
                </select>
              </div>
            </div>
          </div>
        </TransitionGroup>
      </div>

      <div class="pagination-container">
        <div class="pagination" v-show="images.length > pageSize">
          <button class="page-btn" :disabled="currentPage === 1" @click="handlePageChange(currentPage - 1)">
            上一页
          </button>
          <div class="page-numbers">
            <button v-for="pageNum in displayedPages" :key="pageNum" class="page-number" :class="{
              'active': currentPage === pageNum,
              'ellipsis': pageNum === '...'
            }" @click="handlePageNumClick(pageNum)">
              {{ pageNum }}
            </button>
          </div>
          <div class="page-jump">
            <span>跳至</span>
            <input type="number" v-model.number="jumpPage" :min="1" :max="totalPages" @keyup.enter="handleJumpPage"
              @blur="handleJumpPage">
            <span>页</span>
          </div>
          <button class="page-btn" :disabled="currentPage === totalPages" @click="handlePageChange(currentPage + 1)">
            下一页
          </button>
        </div>
      </div>
    </div>

    <div class="compression-notice" v-if="selectedFormat === 'png' || selectedFormat === 'avif'">
      <el-alert :title="formatNoticeTitle" type="info" :description="formatNoticeDesc" show-icon :closable="false" />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, reactive, computed, watch } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'

interface ImageItem {
  file: File
  preview: string
  originalSize: number
  compressedSize: number
  estimatedSize: number
  compressedBlob?: Blob
  dimensions: string
  processing: boolean
  progress: number
  quality: number
  selected: boolean
  estimating: boolean
  outputFormat: string
}

const fileInput = ref<HTMLInputElement | null>(null)
const isDragging = ref(false)
const isUploading = ref(false)
const defaultQuality = ref(75)
const images = reactive<ImageItem[]>([])

// 分页相关的状态
const currentPage = ref(1)
const pageSize = 6

// 计算当前页的图片
const paginatedImages = computed(() => {
  if (images.length === 0) return []
  const start = (currentPage.value - 1) * pageSize
  const end = Math.min(start + pageSize, images.length)
  return images.slice(start, end)
})

// 计算总页数
const totalPages = computed(() => {
  const total = Math.ceil(images.length / pageSize)
  return total > 0 ? total : 1
})

// 切换页码
const handlePageChange = (page: number) => {
  currentPage.value = page
}

// 确保当前页码有效
watch([() => images.length, pageSize], () => {
  const maxPage = Math.ceil(images.length / pageSize)
  if (currentPage.value > maxPage) {
    currentPage.value = Math.max(1, maxPage)
  }
})

// 计算总大小和压缩率
const totalOriginalSize = computed(() =>
  images.reduce((sum, img) => sum + img.originalSize, 0)
)

const totalEstimatedSize = computed(() =>
  images.reduce((sum, img) => sum + img.estimatedSize, 0)
)

const savingRate = computed(() => {
  if (totalOriginalSize.value === 0) return 0
  return Math.round((1 - totalEstimatedSize.value / totalOriginalSize.value) * 100)
})

const savingRateColor = computed(() => {
  if (savingRate.value >= 75) return '#52c41a'
  if (savingRate.value >= 50) return '#1890ff'
  if (savingRate.value >= 25) return '#faad14'
  return '#f5222d'
})

// 添加获取原始索引的方法
const getOriginalIndex = (pageIndex: number): number => {
  return (currentPage.value - 1) * pageSize + pageIndex
}

// 添加格式相关的工具函数
const isLosslessFormat = (format: string): boolean => {
  return format === 'image/png' || format === 'image/avif'
}

// 显示质量控制的条件
const showQualityControl = (format: string): boolean => {
  return !isLosslessFormat(format)
}

// 获取格式说明文本
const getFormatNote = (format: string): string => {
  switch (format) {
    case 'image/png':
      return 'PNG格式为无损压缩，不支持质量调整'
    case 'image/avif':
      return 'AVIF格式为无损压缩，不支持质量调整'
    default:
      return ''
  }
}

// 修改压缩图片方法
const compressImage = async (file: File, quality: number, format: string): Promise<Blob> => {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => {
      const canvas = document.createElement('canvas')
      const ctx = canvas.getContext('2d')

      if (!ctx) {
        reject(new Error('无法获取canvas context'))
        return
      }

      canvas.width = img.width
      canvas.height = img.height
      ctx.drawImage(img, 0, 0)

      // 无损格式不使用质量参数
      const compressionQuality = isLosslessFormat(format) ? undefined : quality / 100

      canvas.toBlob(
        (blob) => {
          if (blob) {
            resolve(blob)
          } else {
            reject(new Error('转换失败'))
          }
        },
        format,
        compressionQuality
      )
    }
    img.onerror = () => reject(new Error('图片加载失败'))
    img.src = URL.createObjectURL(file)
  })
}

// 修改预计大小的计算方法
const calculateEstimatedSize = async (file: File, quality: number, format: string): Promise<number> => {
  try {
    const blob = await compressImage(file, quality, format)
    return blob.size
  } catch (error) {
    console.error('计算预计大小失败:', error)
    const baseSize = file.size

    // 无损格式使用固定比例
    if (isLosslessFormat(format)) {
      const ratio = format === 'image/png' ? 1.2 : 1.0 // PNG可能略大，AVIF保持原大小
      return Math.round(baseSize * ratio)
    }

    // 其他格式使用质量参数
    const qualityRatio = quality / 100
    let formatRatio = 1
    switch (format) {
      case 'image/jpeg':
        formatRatio = 0.8
        break
      case 'image/webp':
        formatRatio = 0.6
        break
    }

    return Math.round(baseSize * qualityRatio * formatRatio)
  }
}

// 修改更新预估大小的方法
const updateEstimate = async (index: number) => {
  if (index >= 0 && index < images.length) {
    const image = images[index]
    image.estimating = true
    try {
      image.estimatedSize = await calculateEstimatedSize(
        image.file,
        image.quality,
        image.outputFormat
      )
    } finally {
      image.estimating = false
    }
  }
}

// 修改更新所有图片预估大小的方法
const updateAllEstimates = async () => {
  for (const [index, image] of images.entries()) {
    image.quality = defaultQuality.value
    await updateEstimate(index)
  }
}

const getImageDimensions = (file: File): Promise<string> => {
  return new Promise((resolve) => {
    const img = new Image()
    img.src = URL.createObjectURL(file)
    img.onload = () => {
      resolve(`${img.width} × ${img.height}`)
      URL.revokeObjectURL(img.src)
    }
  })
}

const checkFile = (file: File): boolean => {
  if (file.size > 20 * 1024 * 1024) {
    ElMessage.error(`文件 ${file.name} 超过20MB限制`)
    return false
  }

  if (!file.type.startsWith('image/')) {
    ElMessage.error(`文件 ${file.name} 不是有效的图片格式`)
    return false
  }

  const isDuplicate = images.some(img =>
    img.file.name === file.name &&
    img.file.size === file.size &&
    img.file.lastModified === file.lastModified
  )

  if (isDuplicate) {
    ElMessage.warning(`文件 ${file.name} 已经添加过了`)
    return false
  }

  return true
}

// 添加最大上传数量限制
const MAX_UPLOAD_COUNT = 18

// 修改文件处理方法
const processFiles = async (files: File[]) => {
  isUploading.value = true

  // 检查当前已有图片数量
  const currentCount = images.length
  const remainingSlots = MAX_UPLOAD_COUNT - currentCount

  if (remainingSlots <= 0) {
    ElMessage.warning('最多只能上传18张图片')
    isUploading.value = false
    return
  }

  // 获取可上传的文件数量
  const filesToProcess = files.slice(0, remainingSlots)
  let successCount = 0

  if (files.length > remainingSlots) {
    ElMessage.warning(`当前还可以上传${remainingSlots}张图片，已自动截取前${remainingSlots}张`)
  }

  for (const file of filesToProcess) {
    if (!checkFile(file)) continue

    try {
      const dimensions = await getImageDimensions(file)
      const imageItem: ImageItem = {
        file,
        preview: URL.createObjectURL(file),
        originalSize: file.size,
        compressedSize: 0,
        estimatedSize: file.size,
        dimensions,
        processing: false,
        estimating: false,
        progress: 0,
        quality: defaultQuality.value,
        selected: false,
        outputFormat: 'image/jpeg' // 始终默认为 JPG 格式
      }

      images.push(imageItem)
      await updateEstimate(images.length - 1)
      successCount++
    } catch (error) {
      console.error('处理文件失败:', error)
      ElMessage.error(`文件 ${file.name} 处理失败`)
    }
  }

  isUploading.value = false

  // 显示上传成功提示
  if (successCount > 0) {
    ElMessage.success({
      message: `成功添加 ${successCount} 张图片`,
      duration: 2000,
      offset: 80,
      customClass: 'upload-success-message'
    })
  }
}

const triggerFileInput = () => {
  if (!isUploadDisabled.value) {
    // 确保文件输入元素存在
    const input = fileInput.value
    if (input) {
      input.value = '' // 清除之前的选择
      input.click()
    }
  }
}

const handleDragEnter = () => {
  isDragging.value = true
}

const handleDragLeave = () => {
  isDragging.value = false
}

const handleDrop = async (e: DragEvent) => {
  try {
    isDragging.value = false
    const files = e.dataTransfer?.files

    if (!files || files.length === 0) return

    if (images.length >= MAX_UPLOAD_COUNT) {
      ElMessage.warning('最多只能上传18张图片')
      return
    }

    await processFiles(Array.from(files))
  } catch (error) {
    console.error('文件处理错误:', error)
    ElMessage.error('文件处理失败，请重试')
  }
}

const handleFileSelect = async (e: Event) => {
  try {
    const input = e.target as HTMLInputElement
    const files = input.files

    if (!files || files.length === 0) return

    if (images.length >= MAX_UPLOAD_COUNT) {
      ElMessage.warning('最多只能上传18张图片')
      return
    }

    await processFiles(Array.from(files))
  } catch (error) {
    console.error('文件处理错误:', error)
    ElMessage.error('文件处理失败，请重试')
  } finally {
    // 确保清空文件输入
    if (fileInput.value) {
      fileInput.value.value = ''
    }
  }
}

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

const getFileExtension = (format: string): string => {
  switch (format) {
    case 'image/jpeg': return '.jpg'
    case 'image/png': return '.png'
    case 'image/webp': return '.webp'
    case 'image/avif': return '.avif'
    default: return '.jpg'
  }
}

const downloadImage = async (index: number) => {
  const originalIndex = getOriginalIndex(index)
  const image = images[originalIndex]
  image.processing = true

  try {
    const blob = await compressImage(image.file, image.quality, image.outputFormat)
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    const fileName = image.file.name.replace(/\.[^/.]+$/, '') + getFileExtension(image.outputFormat)

    link.href = url
    link.download = fileName
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    URL.revokeObjectURL(url)

    image.compressedBlob = blob
    image.compressedSize = blob.size
    ElMessage.success('下载成功')
  } catch (error) {
    console.error('压缩失败:', error)
    ElMessage.error('压缩失败，请重试')
  } finally {
    image.processing = false
  }
}

// 修改上传提示文本
const uploadText = computed(() => {
  if (isDragging.value) return '松开鼠标上传'
  if (isUploading.value) return '正在处理...'
  if (images.length >= MAX_UPLOAD_COUNT) return '已达到最大上传数量'
  return `点击或拖拽图片到这里上传 (${images.length}/${MAX_UPLOAD_COUNT})`
})

// 添加选择相关的状态
const isAllSelected = ref(false)
const selectedCount = computed(() => images.filter(img => img.selected).length)
const hasSelected = computed(() => selectedCount.value > 0)

// 切换全选状态
const toggleSelectAll = () => {
  images.forEach(image => {
    image.selected = isAllSelected.value
  })
}

// 更新全选状态
const updateSelectAll = () => {
  isAllSelected.value = images.length > 0 && images.every(img => img.selected)
}

// 批量下载
const batchDownload = async () => {
  const selectedImages = images.filter(img => img.selected)
  let successCount = 0
  let failCount = 0

  for (const image of selectedImages) {
    image.processing = true
    try {
      const compressedBlob = await compressImage(image.file, image.quality, image.outputFormat)
      const link = document.createElement('a')
      link.href = URL.createObjectURL(compressedBlob)
      link.download = `compressed_${image.file.name}`
      link.click()
      URL.revokeObjectURL(link.href)
      successCount++
    } catch (error) {
      failCount++
      console.error(`压缩失败: ${image.file.name}`, error)
    } finally {
      image.processing = false
    }
  }

  if (successCount > 0) {
    ElMessage.success(`成功下载 ${successCount} 个文件`)
  }
  if (failCount > 0) {
    ElMessage.error(`${failCount} 个文件下载失败`)
  }
}

// 批量删除
const batchDelete = () => {
  ElMessageBox.confirm(
    `确定要删除选中的 ${selectedCount.value} 个文件吗？`,
    '确认删除',
    {
      confirmButtonText: '删除',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    // 从后往前删除，避免索引变化
    for (let i = images.length - 1; i >= 0; i--) {
      if (images[i].selected) {
        // 清理资源
        URL.revokeObjectURL(images[i].preview)
        const blob = images[i].compressedBlob
        if (blob) {
          const objectUrl = URL.createObjectURL(blob)
          URL.revokeObjectURL(objectUrl)
        }
        images.splice(i, 1)
      }
    }
    ElMessage.success('删除成功')
    isAllSelected.value = false
  }).catch(() => {
    // 取消删除
  })
}

// 删除单个图片
const deleteImage = (index: number) => {
  const originalIndex = getOriginalIndex(index)
  ElMessageBox.confirm(
    '确定要删除这张图片吗？',
    '确认删除',
    {
      confirmButtonText: '删除',
      cancelButtonText: '取消',
      type: 'warning'
    }
  ).then(() => {
    // 清理资源
    URL.revokeObjectURL(images[originalIndex].preview)
    const blob = images[originalIndex].compressedBlob
    if (blob) {
      const objectUrl = URL.createObjectURL(blob)
      URL.revokeObjectURL(objectUrl)
    }
    images.splice(originalIndex, 1)
    ElMessage.success('删除成功')
    updateSelectAll()
  }).catch(() => {
    // 取消删除
  })
}

// 添加跳转页码状态
const jumpPage = ref<number | ''>('')

// 处理页码跳转
const handleJumpPage = () => {
  if (typeof jumpPage.value === 'number') {
    const targetPage = Math.min(Math.max(1, jumpPage.value), totalPages.value)
    handlePageChange(targetPage)
    jumpPage.value = ''
  }
}

// 计算要显示的页码
const displayedPages = computed(() => {
  const total = totalPages.value
  const current = currentPage.value
  const arr: (number | string)[] = []

  if (total <= 7) {
    // 如果总页数小于等于7，显示所有页码
    return Array.from({ length: total }, (_, i) => i + 1)
  }

  // 始终显示第一页
  arr.push(1)

  if (current > 4) {
    arr.push('...')
  }

  // 计算中间要显示的页码
  const start = Math.max(2, current - 2)
  const end = Math.min(total - 1, current + 2)

  for (let i = start; i <= end; i++) {
    arr.push(i)
  }

  if (current < total - 3) {
    arr.push('...')
  }

  // 始终显示最后一页
  if (total > 1) {
    arr.push(total)
  }

  return arr
})

// 处理页码点击
const handlePageNumClick = (page: number | string) => {
  if (typeof page === 'number') {
    handlePageChange(page)
  }
}

// 添加上传区域禁用状态
const isUploadDisabled = computed(() => images.length >= MAX_UPLOAD_COUNT)

// 修改图片格式选择器组件
const handleFormatChange = (index: number) => {
  const image = images[index]
  image.quality = defaultQuality.value
  updateEstimate(index)
}

const selectedFormat = ref('')
const formatNoticeTitle = computed(() => {
  return selectedFormat.value === 'png'
    ? 'PNG 格式特性说明'
    : 'AVIF 格式特性说明'
})
const formatNoticeDesc = computed(() => {
  return selectedFormat.value === 'png'
    ? '当前选择的 PNG 格式采用无损压缩技术，系统将自动优化处理以确保最佳图像质量'
    : '当前选择的 AVIF 格式采用无损压缩技术，系统将自动优化处理以确保最佳图像质量'
})
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  color: #333;
  position: relative;
  z-index: 1;
  animation: fadeIn 0.5s ease-out;
}

.title {
  text-align: center;
  font-size: 2.5rem;
  background: linear-gradient(120deg, #2196f3, #00bcd4, #1976d2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 3rem;
  font-weight: 700;
  text-shadow: 0 2px 10px rgba(33, 150, 243, 0.3);
  animation: titleGlow 3s ease-in-out infinite;
}

@keyframes titleGlow {
  0%,
  100% {
    filter: brightness(100%) blur(0);
  }

  50% {
    filter: brightness(110%) blur(1px);
  }
}

.upload-container {
  margin-bottom: 2rem;
}

#drop-zone {
  position: relative;
  border: 2px dashed #e0e0e0;
  border-radius: 20px;
  padding: 3.5rem;
  text-align: center;
  background: rgba(255, 255, 255, 0.95);
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(10px);
  overflow: hidden;
}

#drop-zone::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg,
      transparent,
      rgba(33, 150, 243, 0.1),
      transparent);
  transform: rotate(45deg);
  animation: shimmer 3s infinite;
  pointer-events: none;
}

@keyframes shimmer {
  0% {
    transform: translateX(-100%) rotate(45deg);
  }

  100% {
    transform: translateX(100%) rotate(45deg);
  }
}

.upload-icon {
  color: #2196f3;
  margin-bottom: 1rem;
  animation: float 3s ease-in-out infinite;
}

.upload-icon.uploading {
  animation: spin 1s linear infinite;
}

.upload-text {
  font-size: 1.1rem;
  color: #333;
  margin: 1rem 0;
  transition: color 0.3s ease;
}

#drop-zone:hover .upload-text {
  color: #2196f3;
}

.upload-tip {
  display: block;
  margin-top: 0.5rem;
  color: #666;
  font-size: 0.875rem;
}

.compression-settings {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 16px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
}

.batch-operations {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 8px;
}

.checkbox-wrapper {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.select-all-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  user-select: none;
  padding: 6px 12px;
  border-radius: 6px;
  transition: all 0.3s ease;
  background: #f5f7fa;
}

.select-all-label:hover {
  background: #ecf5ff;
}

.checkbox-text {
  font-size: 14px;
  color: #606266;
  font-weight: 500;
}

.select-all-checkbox {
  appearance: none;
  width: 16px;
  height: 16px;
  border: 2px solid #dcdfe6;
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  transition: all 0.3s ease;
  background-color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}

.select-all-checkbox:checked {
  background-color: var(--el-color-primary);
  border-color: var(--el-color-primary);
}

.select-all-checkbox:checked::after {
  content: '';
  position: absolute;
  width: 4px;
  height: 8px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
  margin-top: -2px;  /* 微调垂直位置 */
}

.image-checkbox {
  position: absolute;
  top: 12px;
  left: 12px;
  z-index: 2;
  width: 24px;
  height: 24px;
  opacity: 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  border: 2px solid rgba(255, 255, 255, 0.9);
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.image-checkbox:checked {
  background: var(--el-color-primary);
  border-color: var(--el-color-primary);
}

.image-checkbox:checked::after {
  content: '';
  position: absolute;
  width: 6px;
  height: 12px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
  margin-top: -2px;  /* 微调垂直位置 */
}

.image-item:hover .image-checkbox {
  opacity: 1;
}

.image-item.selected .image-checkbox {
  opacity: 1;
}

.delete-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  width: 24px;
  height: 24px;
  background: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
  opacity: 0;
  transition: all 0.3s ease;
}

.image-item:hover .delete-btn {
  opacity: 1;
}

.delete-btn:hover {
  background: #ff4d4f;
  color: white;
}

.delete-icon {
  font-size: 18px;
  line-height: 1;
}

.batch-btn {
  padding: 0.625rem 1rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  color: white;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: 500;
  font-size: 0.875rem;
  position: relative;
  overflow: hidden;
}

.batch-btn::before {
  content: '';
  display: inline-block;
  width: 14px;
  height: 14px;
  background-size: contain;
  background-repeat: no-repeat;
}

.batch-btn.download {
  background: #2196f3;
}

.batch-btn.download:hover {
  background: #1976d2;
}

.batch-btn.delete {
  background: #ff4d4f;
}

.batch-btn.delete:hover {
  background: #f5222d;
}

.image-item.selected {
  outline: 2px solid #2196f3;
  box-shadow: 0 0 0 4px rgba(33, 150, 243, 0.1);
}

.right-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

.selected-count {
  display: inline-flex;
  align-items: center;
  padding: 4px 12px;
  margin-left: 12px;
  font-size: 13px;
  color: var(--el-color-primary);
  background: rgba(64, 158, 255, 0.1);
  border-radius: 20px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.count-number {
  display: inline-block;
  margin: 0 4px;
  font-size: 15px;
  font-weight: 600;
  color: var(--el-color-primary);
  min-width: 16px;
  text-align: center;
  font-feature-settings: "tnum";
  animation: countPulse 0.3s ease-out;
}

@keyframes countPulse {
  0% {
    transform: scale(0.8);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.selected-count:hover {
  background: rgba(64, 158, 255, 0.15);
  transform: translateY(-1px);
}

.image-item.selected {
  outline: 2px solid #2196f3;
  box-shadow: 0 0 0 4px rgba(33, 150, 243, 0.1);
}

.total-stats {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-item span {
  font-size: 0.875rem;
  color: #666;
}

.stat-item strong {
  font-size: 1.25rem;
  color: #2196f3;
}

.arrow {
  color: #666;
  font-size: 1.5rem;
}

.saving-rate {
  padding: 0.25rem 0.75rem;
  border-radius: 12px;
  background: var(--saving-color);
  color: white;
  font-weight: 600;
  font-size: 0.875rem;
}

.image-item {
  position: relative;
  background: rgba(255, 255, 255, 0.98);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.06);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.image-item:hover {
  transform: translateY(-4px) scale(1.01);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 0.5rem;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  font-size: 0.875rem;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-item:hover .image-overlay {
  opacity: 1;
}

.file-name {
  font-weight: 500;
  margin-bottom: 0.5rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.compression-progress {
  height: 4px;
  background: #eee;
  border-radius: 2px;
  overflow: hidden;
  margin: 0.5rem 0;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(45deg, #2196f3, #00bcd4);
  transition: width 0.3s ease;
}

.spinner {
  width: 16px;
  height: 16px;
  border: 2px solid #fff;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

.quality-control {
  margin: 2rem 0;
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  border: 1px solid #ebeef5;
}

.quality-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.quality-title {
  font-size: 15px;
  font-weight: 600;
  color: #303133;
}

.quality-tip {
  font-size: 12px;
  color: #909399;
}

.slider-container {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 8px 0;
}

.slider {
  flex: 1;
  -webkit-appearance: none;
  height: 6px;
  background: linear-gradient(90deg, 
    var(--el-color-primary) var(--value, 0%), 
    #e4e7ed var(--value, 0%)
  );
  border-radius: 3px;
  outline: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  background: white;
  border: 2px solid var(--el-color-primary);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
  box-shadow: 0 2px 8px rgba(64, 158, 255, 0.3);
}

.quality-value {
  min-width: 48px;
  padding: 4px 12px;
  text-align: center;
  background: #f5f7fa;
  border-radius: 4px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.quality-value .number {
  font-family: 'SF Mono', Monaco, Menlo, Consolas, 'Courier New', monospace;
  font-size: 16px;
  font-weight: 700;
  color: var(--el-color-primary);
  letter-spacing: -0.5px;
}

.percent-sign {
  font-size: 12px;
  font-weight: 500;
  opacity: 0.8;
  margin-left: 2px;
  color: var(--el-color-primary);
}

.image-list {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.25rem;
  min-height: 400px;
  max-width: 1200px;
  margin: 0 auto;
}

.image-preview {
  position: relative;
  padding-top: 75%;
  width: 100%;
  overflow: hidden;
}

.image-preview img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.image-info {
  padding: 1.25rem;
  height: auto;
}

.size-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin: 0.75rem 0;
  padding: 0.75rem;
  background: #f8f9fa;
  border-radius: 12px;
}

.size-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  text-align: center;
}

/* 原始大小的样式 */
.size-item:first-child strong {
  font-size: 1.1rem;
  color: #666;
  font-weight: 600;
}

/* 预计大小的样式 */
.size-item:last-child strong {
  font-size: 1.1rem;
  color: #00bcd4;
  /* 使用更清爽的蓝绿色 */
  font-weight: 600;
  background: linear-gradient(45deg, #00bcd4, #2196f3);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 2px 4px rgba(0, 188, 212, 0.1);
}

/* 计算中状态的样式 */
.size-item:last-child strong:empty::before {
  content: '计算中...';
  color: #909399;
  background: none;
  -webkit-text-fill-color: currentColor;
}

.size-arrow {
  display: flex;
  align-items: center;
  color: #666;
  font-size: 1.25rem;
  position: relative;
  width: 40px;
  justify-content: center;
}

.size-arrow::before {
  content: '';
  position: absolute;
  width: 24px;
  height: 2px;
  background: currentColor;
  transform-origin: right;
  animation: arrowSlide 2s infinite;
}

.size-arrow::after {
  content: '';
  position: absolute;
  right: 4px;
  width: 8px;
  height: 8px;
  border-right: 2px solid currentColor;
  border-top: 2px solid currentColor;
  transform: rotate(45deg);
  animation: arrowPoint 2s infinite;
}

@keyframes arrowSlide {
  0% {
    transform: scaleX(0.3);
    opacity: 0;
  }

  50% {
    transform: scaleX(1);
    opacity: 1;
  }

  100% {
    transform: scaleX(0.3);
    opacity: 0;
  }
}

@keyframes arrowPoint {
  0% {
    opacity: 0;
  }

  50% {
    opacity: 1;
  }

  100% {
    opacity: 0;
  }
}

.download-btn {
  width: 100%;
  padding: 0.85rem;
  background: linear-gradient(45deg, #2196f3, #00bcd4);
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-weight: 600;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.download-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg,
      transparent,
      rgba(255, 255, 255, 0.2),
      transparent);
  transition: 0.5s;
}

.download-btn:hover::before {
  left: 100%;
}

.image-list-move,
.image-list-enter-active,
.image-list-leave-active {
  transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.image-list-enter-from,
.image-list-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.image-list-leave-active {
  position: absolute;
}

.images-container {
  margin-bottom: 2rem;
}

.pagination-container {
  width: 100%;
  margin-top: 2rem;
  padding: 1rem 0;
  display: flex;
  justify-content: center;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  padding: 1.2rem;
  background: transparent;
  box-shadow: none;
  border-radius: 16px;
  min-height: 60px;
}

.page-numbers {
  display: flex;
  gap: 0.5rem;
}

.page-btn,
.page-number {
  min-width: 40px;
  height: 40px;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: none;
  background: transparent;
  color: #666;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.page-btn:hover:not(:disabled),
.page-number:hover:not(.active) {
  background: transparent;
  color: #2196f3;
  transform: translateY(-1px);
}

.page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  box-shadow: none;
}

.page-number.active {
  background: #2196f3;
  color: white;
  box-shadow: 0 2px 6px rgba(33, 150, 243, 0.3);
}

/* 优化跳转页码输入框 */
.page-jump {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0 16px;
  color: #666;
  font-size: 14px;
}

.page-jump input {
  width: 50px;
  height: 32px;
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  text-align: center;
  transition: all 0.3s;
  background: transparent;
  /* 隐藏上下箭头 */
  -webkit-appearance: none;
  -moz-appearance: textfield;
}

/* 隐藏 Chrome/Safari/Edge 的上下箭头 */
.page-jump input::-webkit-inner-spin-button,
.page-jump input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* 隐藏 Firefox 的上下箭头 */
.page-jump input[type=number] {
  -moz-appearance: textfield;
}

.page-jump input:focus {
  outline: none;
  background: rgba(33, 150, 243, 0.1);
}

/* 添加输入框底部边框 */
.page-jump input {
  border-bottom: 1px solid #e0e0e0;
}

.page-jump input:focus {
  border-bottom-color: #2196f3;
}

@media (max-width: 1200px) {
  .image-list {
    grid-template-columns: repeat(2, 1fr);
  }

  .image-item {
    max-width: 320px;
  }
}

@media (max-width: 768px) {
  .image-list {
    grid-template-columns: repeat(1, 1fr);
  }

  .image-item {
    max-width: 300px;
  }
}

.file-input {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
  opacity: 0;
}

:deep(.el-message) {
  z-index: 9999;
}

:deep(.el-message--error) {
  background-color: #fef0f0;
  border-color: #fde2e2;
}

:deep(.el-message--warning) {
  background-color: #fdf6ec;
  border-color: #faecd8;
}

.individual-quality-control {
  margin: 1rem 0;
  padding: 0.75rem;
  background: #f5f5f5;
  border-radius: 8px;
}

.individual-quality-control label {
  display: block;
  margin-bottom: 0.5rem;
  font-size: 0.875rem;
  color: #666;
}

.individual-quality-control .slider-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

/* 添加页码跳转样式 */
.page-jump {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0 16px;
  color: #666;
  font-size: 14px;
}

/* 优化页码按钮样式 */
.page-number.ellipsis {
  background: transparent;
  border: none;
  cursor: default;
  pointer-events: none;
}

.page-number.ellipsis:hover {
  background: transparent;
  transform: none;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .page-jump {
    width: 100%;
    justify-content: center;
    margin: 8px 0;
  }

  .pagination {
    padding: 8px;
  }
}

/* 添加禁用状态样式 */
#drop-zone.disabled {
  cursor: not-allowed;
  opacity: 0.7;
  border-color: #d9d9d9;
  background: #f5f5f5;
}

#drop-zone.disabled:hover {
  transform: none;
  border-color: #d9d9d9;
  box-shadow: none;
}

#drop-zone.disabled .upload-icon {
  color: #999;
}

#drop-zone.disabled .upload-text {
  color: #999;
}

/* 添加计数器样式 */
.upload-counter {
  position: absolute;
  top: 1rem;
  right: 1rem;
  font-size: 0.875rem;
  color: #666;
  background: rgba(255, 255, 255, 0.9);
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
}

/* 添加成功提示样式 */
:deep(.upload-success-message) {
  background-color: #f0f9eb !important;
  border-color: #e1f3d8 !important;
  padding: 12px 20px !important;
}

:deep(.upload-success-message .el-message__content) {
  color: #67c23a !important;
  font-weight: 500;
}

:deep(.upload-success-message .el-message__icon) {
  color: #67c23a !important;
  font-size: 16px;
}

.format-control {
  margin: 0.5rem 0;
  padding: 0.75rem;
  background: #f5f5f5;
  border-radius: 8px;
}

.format-control label {
  display: block;
  margin-bottom: 0.5rem;
  font-size: 0.875rem;
  color: #666;
}

.format-control select {
  width: 100%;
  height: 40px;
  padding: 0 0.75rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  background: white;
  color: #333;
  font-size: 0.875rem;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-weight: 500;
}

.format-control select:hover {
  border-color: #2196f3;
  transform: translateY(-1px);
}

.format-control select:focus {
  outline: none;
  border-color: #2196f3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.2);
}

/* 优化下拉选择器样式 */
.format-control select {
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%23666' d='M6 8.825L1.175 4 2.238 2.938 6 6.7l3.763-3.762L10.825 4z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 0.75rem center;
  padding-right: 2rem;
}

.format-note {
  margin: 0.5rem 0;
  padding: 0.75rem;
  background: #f5f5f5;
  border-radius: 8px;
  color: #666;
  font-size: 0.875rem;
  text-align: center;
}

.compression-notice {
  margin: 16px 0;
}

/* 自定义 Alert 样式 */
:deep(.el-alert) {
  background: linear-gradient(to right, #f8f9fa, #ffffff);
  border: 1px solid #e4e7ed;
  border-radius: 8px;
  padding: 12px 16px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.02);
  transition: all 0.3s ease;
}

:deep(.el-alert:hover) {
  transform: translateY(-1px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  border-color: #dcdfe6;
}

:deep(.el-alert--info) {
  border-left: 4px solid #909399;
}

:deep(.el-alert__icon) {
  font-size: 16px;
  color: #909399;
  margin-right: 12px;
}

:deep(.el-alert__title) {
  font-size: 14px;
  font-weight: 500;
  color: #303133;
  line-height: 1.6;
}

:deep(.el-alert__description) {
  margin: 4px 0 0;
  font-size: 13px;
  color: #606266;
  line-height: 1.6;
}

/* 添加动画效果 */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.compression-notice {
  animation: slideIn 0.3s ease-out;
}

/* 添加全局过渡动画 */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 添加加载动画 */
@keyframes pulse {
  0% {
    transform: scale(0.95);
    opacity: 0.7;
  }

  50% {
    transform: scale(1);
    opacity: 1;
  }

  100% {
    transform: scale(0.95);
    opacity: 0.7;
  }
}

.image-item.processing {
  animation: pulse 1.5s ease-in-out infinite;
}

/* Ant Design 风格按钮样式 */
.ant-btn {
  display: inline-flex !important;
  align-items: center;
  gap: 6px !important;
  height: 32px;
  padding: 4px 16px !important;
  font-size: 14px;
  border-radius: 6px !important;
  transition: all 0.2s cubic-bezier(0.645, 0.045, 0.355, 1) !important;
}

.ant-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.ant-btn:active {
  transform: translateY(0);
}

/* Ant Design 图标样式 */
.anticon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 16px;
  height: 16px;
  line-height: 0;
  vertical-align: -0.125em;
  transition: transform 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
  margin-right: 4px;
}

.ant-btn:hover .anticon {
  transform: scale(1.15) rotate(-4deg);
}

.ant-btn[type="danger"]:hover .anticon {
  transform: scale(1.15) rotate(4deg);
}

/* 主要按钮样式 */
.ant-btn[type="primary"] {
  border-color: var(--el-color-primary) !important;
  color: var(--el-color-primary) !important;
}

.ant-btn[type="primary"]:hover {
  background: var(--el-color-primary-light-9) !important;
  border-color: var(--el-color-primary) !important;
}

/* 危险按钮样式 */
.ant-btn[type="danger"] {
  border-color: var(--el-color-danger) !important;
  color: var(--el-color-danger) !important;
}

.ant-btn[type="danger"]:hover {
  background: var(--el-color-danger-light-9) !important;
  border-color: var(--el-color-danger) !important;
}

/* 禁用状态 */
.ant-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed !important;
  transform: none !important;
  box-shadow: none !important;
}

.ant-btn:disabled .anticon {
  transform: none !important;
}

/* 按钮内容样式 */
.ant-btn span {
  font-weight: 500;
  letter-spacing: 0.3px;
  position: relative;
  top: 0.5px;
  margin-left: 2px;
}

/* 右侧操作按钮容器 */
.right-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}
</style>